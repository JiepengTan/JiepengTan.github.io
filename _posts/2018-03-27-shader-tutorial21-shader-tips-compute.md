---
layout: post
title:  "中级Shader教程21 优化:用shader分摊CPU压力"
date:   2018-04-25 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial theory shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
<p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/JobSys/head.gif?raw=true" width="256"></p>

### **粒子大规模渲染优化** 
### **0.优化总览**

大规模粒子渲染，和动画系统优化一样，需要优化和权衡的是GPU和CPU的负载，同样技巧也相似。

**动画系统的优化**  
1.DrawCall的减少  
　　　Batch  单个模型顶点数量太多，可以合批的数量不多  
　　　GPUInstance 手游限制  

2.负载的均衡  
　　　CPU:可以通过预计算缓存的方式来优化如插件[MeshAnimator][4]  
　　　GPU:[GPUSkin][5]的方式进行优化  

**粒子系统的优化**  
1.DrawCall的减少  
　　　Batch  单个模型顶点数量少，手游重点优化范式  
　　　GPUInstance 手游限制  
2.负载的均衡  
　　　CPU:在Unity2018版本中可以使用ECS+JobSystem这个大杀器  
　　　GPU:使用shader来批量计算针对单个粒子的信息如位置，旋转，生命期等  
       






### **1.DrawCall优化**
#### 1.粒子系统
通过引擎原生的粒子系统来发射粒子，然后通过脚本控制每个粒子的位置和旋转，引擎自动将响应的粒子进行合批优化。  
#### 2.GPU Instance
自己手动模拟粒子系统来控制粒子的轨迹，通过GPU instance 将众多的粒子进行批量渲染  
#### 3.合批
将众多的小模型进行拼接到一个大模型中，然后将模型id写入到顶点信息中，在渲染过程中:  
1.cpu计算粒子的移动的信息，将其写入到一张贴图A中  
2.在shader中通过模型信息中获取粒子ID,然后通过ID从贴图A获取粒子的TRS矩阵，以及其他的信息（如生命期），渲染之.  

整体流程如下：
>update 
>1.计算粒子信息  
>2.将信息写入贴图  
>3.将数量，偏移等信息提交渲染  

### **2.运算优化**

#### 0.整个框架
0.Editor  
　　　将众多小模型合并为一个大模型，并在顶点信息中写入ID  
1.Init  
　　　初始化 创建贴图 FilterMode.Point，TextureWrapMode.Clamp  
2.Update data info  
　　　**不同的方案在这里不同**  
3.Render in shader  
　　　1.从顶点信息中获取所属particle的ID  
　　　2.通过ID计算所需信息在贴图中的下标  
　　　3.获取并应用该信息  

范例shader：
```c
sampler2D _TextureMatrix;//旋转变化贴图信息
uniform float4 _TexIdxInfo;//贴图大小 x width y heigh z 当前合批的Mesh在开始索引下标

struct appdata
{
    float4 vertex : POSITION;
    fixed4 color : COLOR;
    float2 uv : TEXCOORD0;
    float2 uv1 : TEXCOORD1;//记录粒子的ID
};
// 通过粒子ID 计算信息在贴图的位置
inline float4 indexToUV(float index)
{
    int row = (int)(index / _TexIdxInfo.x);
    float col = index - row * _TexIdxInfo.x;
    float x = col / _TexIdxInfo.x;
    float y = row / _TexIdxInfo.y;
    return float4(x,y, 0, 0);
}
inline float4 getPosOffset(int idxOffset, int meshStartIdx, float instanceIdx)
{
    float matStartIndex = idxOffset  + meshStartIdx + instanceIdx;
    float4 row0 = tex2Dlod(_TextureMatrix, indexToUV(matStartIndex));
    return row0;
}
v2f vert( appdata v) 
{
    // 获取偏移
    float4 offset = getPosOffset(_TexIdxInfo.w * 0, _TexIdxInfo.z, v.uv1.x); 
    float4 pos= float4(offset.xyz,0) + v.vertex;    
    v2f o;
    o.vertex = UnityObjectToClipPos(pos); 
    o.color = v.color;
    o.uv = TRANSFORM_TEX(v.uv, _MainTex);
    return o;
}

```

#### 1.主线程计算
简单遍历所有的粒子，更新其轨迹信息  
这个简单就不贴代码了。  

#### 2.ECS+JobSystem 加速计算
将整个粒子轨迹信息的更新步骤进行细致的拆分，利用Unity 2018 新的ECS+jobSystem来加速，将所有的计算多线程化，基本上，cpu有多少核，性能就能提升几倍(JobSystem)，甚至更多（ECS 数据紧凑，内存缓存友好）  
范例如下：  
```cs
//20180413 create by jiepengtan email: jiepengtan@gamil.com
public class ScriptableParticleEffect_JobSys : MonoBehaviour
{
    public MeshTemplate templete;
    public Mesh model;
    public float radius;
    public int particleNum;
    public float minSpd;
    public float maxSpd;
    public Material defaultMat;
    public float loopTime = 10;//循环时间

    public int realParticleNum;
    public int meshNum;
    NativeArray<Vector3> srcPos;
    NativeArray<Vector3> dstPos;
    NativeArray<Vector3> curPos;
    NativeArray<Vector2> randSpd;//x spd y maxTime
    NativeArray<Color> texInfo0;
    NativeArray<Color> texInfo1;
    NativeArray<Color> texInfo2;

    List<MeshRenderer> renders = new List<MeshRenderer>();
    public float timer;
    public Texture2D texMatrix;
    public int texWid;
    public int texHei;
    public const int pixelPerUnit = 3;
    public int totalPixelNum { get { return texWid * texHei; } }
    Color[] tempColorArray;
    JobHandle m_PositionJobHandle;
    bool isFirstUpdate = true;
    void Start()
    {
        if (model == null || templete == null)
        {
            this.enabled = false;
            return;
        }
        meshNum = Mathf.CeilToInt(particleNum * 1.0f / templete.instanceCount);
        realParticleNum = meshNum * templete.instanceCount;
        //create texture
        CreateTexture();
        //创建Mesh
        CreateRenders();
        var vetices = model.vertices;
        //根据目标数量生成相应的srcPos 和targetPos
        srcPos = new NativeArray<Vector3>(realParticleNum, Allocator.Persistent);
        dstPos = new NativeArray<Vector3>(realParticleNum, Allocator.Persistent);
        curPos = new NativeArray<Vector3>(realParticleNum, Allocator.Persistent);
        randSpd = new NativeArray<Vector2>(realParticleNum, Allocator.Persistent);
        texInfo0 = new NativeArray<Color>(realParticleNum, Allocator.Persistent);
        texInfo1 = new NativeArray<Color>(realParticleNum, Allocator.Persistent);
        texInfo2 = new NativeArray<Color>(realParticleNum, Allocator.Persistent);
        tempColorArray = new Color[totalPixelNum];
        var vetCount = vetices.Length;
        //init pos
        for (int i = 0; i < realParticleNum; i++)
        {
            dstPos[i] = vetices[Random.Range(0, vetCount)];
            var len = Random.value * radius;
            var deg = Random.value * Mathf.PI * 2.0f;
            var x = Mathf.Cos(deg) * len;
            var z = Mathf.Sin(deg) * len;
            srcPos[i] = new Vector3(x, 0.1f, z);
            curPos[i] = srcPos[i];
            var spd = Random.Range(minSpd, maxSpd);
            var time = Vector3.Distance(dstPos[i], srcPos[i]) / spd;
            randSpd[i] = new Vector2(spd, time);
        }
    }


    struct PositonJob : IJobParallelFor
    {
        [ReadOnly]
        public NativeArray<Vector3> srcPos;
        [ReadOnly]
        public NativeArray<Vector3> dstPos;
        [ReadOnly]
        public NativeArray<Vector2> randSpd;
        public NativeArray<Vector3> curPos;
        public float timer;

        public void Execute(int i)
        {
            var sp = srcPos[i];
            var dp = dstPos[i];
            var progrss = math.clamp(randSpd[i].x * timer / math.distance(sp, dp), 0.0f, 1.0f);
            var pos = math.lerp(sp, dp, progrss);
            curPos[i] = pos;
        }
    }
    struct MatrixTexJob : IJobParallelFor
    {
        [ReadOnly]
        public NativeArray<Vector3> curPos;
        public NativeArray<Color> texInfo0;
        public NativeArray<Color> texInfo1;
        public NativeArray<Color> texInfo2;
        public void Execute(int i)
        {
            var pos = curPos[i];
            //...计算ratate 等信息 TODO
            texInfo0[i] = new Color(pos.x, pos.y, pos.z, 1.0f);
        }
    }

    public void Update(){
        if (!isFirstUpdate){
            m_PositionJobHandle.Complete();
            UpdateRenderInfo();
        }
        isFirstUpdate = false;
        timer += Time.deltaTime;
        timer = timer % loopTime;
        var m_AccelJob = new PositonJob(){
            timer = timer,
            srcPos = srcPos,
            dstPos = dstPos,
            randSpd = randSpd,
            curPos = curPos
        };
        var m_Job = new MatrixTexJob() {
            texInfo0 = texInfo0,
            texInfo1 = texInfo1,
            texInfo2 = texInfo2,
            curPos = curPos
        };
        var m_AccelJobHandle = m_AccelJob.Schedule(realParticleNum, 64);
        m_PositionJobHandle = m_Job.Schedule(realParticleNum, 64, m_AccelJobHandle);
    }
    void UpdateRenderInfo(){
        texInfo0.ToArray().CopyTo(tempColorArray, realParticleNum * 0);
        texInfo1.ToArray().CopyTo(tempColorArray, realParticleNum * 1);
        texInfo2.ToArray().CopyTo(tempColorArray, realParticleNum * 2);
        texMatrix.SetPixels(tempColorArray);
        texMatrix.Apply();
        for (int i = 0; i < renders.Count; i++){
            var mat = renders[i].sharedMaterial;
            mat.SetTexture("_TextureMatrix", texMatrix);
            mat.SetVector("_TexIdxInfo", new Vector4(texWid, texHei, i * templete.instanceCount, totalPixelNum));
        }
    }
    private void CreateTexture(){
        texWid = 2;
        texHei = 2;
        var totalNum = realParticleNum * pixelPerUnit;
        while (texWid * texHei < totalNum){
            texWid *= 2;
            if (texWid * texHei < totalNum){
                texHei *= 2;
            }
        }
        texMatrix = new Texture2D(texWid, texHei, TextureFormat.RGBAFloat, true);
        texMatrix.filterMode = FilterMode.Point;
        texMatrix.wrapMode = TextureWrapMode.Clamp;
    }

    void CreateRenders()
    {
        for (int i = 0; i < meshNum; i++)
        {
            var go = new GameObject();
            go.transform.SetParent(transform);
            var filter = go.AddComponent<MeshFilter>();
            filter.sharedMesh = templete.mesh;
            var render = go.AddComponent<MeshRenderer>();
            render.sharedMaterial = new Material(defaultMat);
            renders.Add(render);
        }
    }
    void OnDestroy()
    {
        m_PositionJobHandle.Complete();
        srcPos.Dispose();
        //... destroy other
    }
}
```
    
#### 3.GPU计算  

1.准备计算所需的数据  
2.通过Graphics.Blit的方式使用shader进行运算，计算结果在RenderTexture中  
3.目标shader利用上述计算结果进行运算 具体操作类似  

与上面有所不同的是Update的部分：  

```c
//1.设置属性
RenderTexture prePosBuffer;
RenderTexture preVelBuffer;
RenderTexture posBuffer;
RenderTexture velBuffer;
RenderTexture rotBuffer;
...
_material.SetVector("_Gravity", _gravity * Time.deltaTime);
_material.SetVector("_Life", new Vector2(1.,1.));
_material.SetTexture("_PositionBuffer", prePosBuffer); 
_material.SetTexture("_VelocityBuffer", preVelBuffer);
...

//2.渲染
Graphics.Blit(null, posBuffer, _material, 0);
...

//3.将计算的结果 赋予目标shader
MaterialPropertyBlock block =  = new MaterialPropertyBlock();;
block.SetTexture("_PositionBuffer", posBuffer);
block.SetTexture("_VelocityBuffer", velBuffer);
block.SetTexture("_RotationBuffer", rotBuffer);
meshRenderer.SetPropertyBlock(_propertyBlock);

//4.如果有先后依赖关系的计算，可以使用双buff
SwapBuffer();
```


  ## [**配套视频**][40]  
- [本教程配套blog ][1]
  - [本教程配套项目源码 ][2]
  - [教程中抽取的RayMarching框架][3]


  [1]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [2]: https://github.com/JiepengTan/FishManShaderTutorial
  [40]:https://space.bilibili.com/308864667/channel/detail?cid=112754
  [3]: https://github.com/JiepengTan/Unity-Raymarching-Framework
  [4]: https://www.assetstore.unity3d.com/#!/content/26009
  [5]: https://github.com/chengkehan/GPUSkinning