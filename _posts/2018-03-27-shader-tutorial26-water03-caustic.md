---
layout: post
title:  "中级Shader教程26 水专题03:三种Caustic实现方式"
date:   2018-04-26 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial theory shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Caustic/head.gif?raw=true" width="768"></p>




### 0.提要
    Caustic的的特征是高光集中而且扭动，模拟的是由于水面的扭动导致阳光的折射集中产生的效果，
如图，本篇讲述的是三种模拟算法，并非由光线追踪精确计算得到。  

```c
v2f vert (appdata v){
    v2f o;
    o.vertex = UnityObjectToClipPos(v.vertex);
    o.uv = TRANSFORM_TEX(v.uv, _MainTex);
    return o;
}
fixed4 frag (v2f i) : SV_Target{
    float2 uv = _TileNum * i.uv;
    float time = _Time.y;
    float val = CausticRotateMin(uv,time);//替换相应的函数即可
    return float4(val,val,val,1.0);//+float4(0,.35,.5,1);
}
```
### 1.基于多层三角函数的扭动叠加
上面第1种效果  
```c
float3 CausticTriTwist(float2 uv,float time )
{
    const int MAX_ITER = 5;
    float2 p = fmod(uv*PI2,PI2 )-250.0;//1.空间划分

    float2 i = float2(p);
    float c = 1.0;
    float inten = .005;

    for (int n = 0; n < MAX_ITER; n++) //3.多层叠加
    {
        float t = time * (1.0 - (3.5 / float(n+1)));
        i = p + float2(cos(t - i.x) + sin(t + i.y), sin(t - i.y) + cos(t + i.x));//2.空间扭曲
        c += 1.0/length(float2(p.x / (sin(i.x+t)/inten),p.y / (cos(i.y+t)/inten)));//集合操作avg
    }
    
    c /= float(MAX_ITER);
    c = 1.17-pow(c, 1.4);//4.亮度调整
    float val = pow(abs(c), 8.0);
    return val;
}
```
基本思想是：   
1.先实现grid划分，调节到想要的亮度分布 如图一  
2.实现空间扭曲 如图二  
3.多层数值叠加 如图三    
4.亮度调整 配色   

<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Caustic/caustic_tri0.jpg?raw=true" width="256"> <img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Caustic/caustic_tri1.gif?raw=true" width="256"> <img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Caustic/caustic_tri2.gif?raw=true" width="256">

### 2.基于连续空间旋转min叠加
上面第3种效果  
基本思想是： 
1.将uv空间缩放得到一个合适密度大小的空间  
2.对单独每一各纬度的空间进行旋转缩放以及偏移  
3.多重空间的值进行操作如length 或者其他的计算  
4.重复2，3操作多次  
5.多层混合的值进行min操作  
6.亮度调整，颜色调整    
注意：  
这里的步骤2，3中的旋转矩阵的行列式det要比较接近于1(0.2~5之间会比较好)，因为在在数学上行列式是一个线性变换对“体积”所造成的影响，所以在对一个相对均匀的空间进行矩阵操作中，不会出现过于大的拉伸，这样得到的整体效果的分布会比较均匀。  

图片顺序从左到右，从上到下  
图 1，2，3中是矩阵操作后整个颜色空间的运动情况  
图5，6 是对颜色空间的length解释  
图7是多重矩阵操作之间颜色空间的叠加min后的效果  

源码：
```c
float CausticRotateMin(float2 uv, float time){
    float3x3 mat = float3x3(2,1,-2, 3,-2,1, 1,2,2);//1.2.操作矩阵
    float3 vec1 = mul(mat*0.5,float3(uv,time));//3.对颜色空间进行操作
    float3 vec2 = mul(mat*0.4,vec1);//4.重复2，3操作
    float3 vec3 = mul(mat*0.3,vec2);
    float val = min(length(frac(vec1)-0.5),length(frac(vec2)-0.5));//5.集合操作 min
    val = min(val,length(frac(vec3)-0.5));
    val = pow(val,7.0)*25.;//6.亮度调整
    return val;
}

```


<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Caustic/caustic_rot1.gif?raw=true" width="256"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Caustic/caustic_rot2.gif?raw=true" width="256"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Caustic/caustic_rot3.gif?raw=true" width="256">

<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Caustic/caustic_rot4.gif?raw=true" width="256"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Caustic/caustic_rot5.gif?raw=true" width="256"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Caustic/caustic_rot6.gif?raw=true" width="256">

<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Caustic/caustic_rot7.gif?raw=true" width="256">


### 3.基于Voronoi算法
voronoi的基本效果图如下：    
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/Base/Voronoi/head.gif?raw=true" width="256">  

本身就很类似水底光影效果了。如果让这边缘变化更加丰富点就更好了，voronoi本身也是noise算法的一种，所以我们直接可以联想到FBM,基于这个想法就得到了上面第二种效果  
源码：
```c
float2 hash22(float2 p){
    float3 p3 = frac(float3(p.xyx) * HASHSCALE3);
    p3 += dot(p3, p3.yzx+19.19);
    return frac((p3.xx+p3.yz)*p3.zy);
}
//voronoi 算法
float wnoise(float2 p,float time) {
    float2 n = floor(p);
    float2 f = frac(p);
    float md = 5.0;
    float2 m = float2(0.,0.);
    for (int i = -1;i<=1;i++) {
        for (int j = -1;j<=1;j++) {
            float2 g = float2(i, j);
            float2 o = hash22(n+g);
            o = 0.5+0.5*sin(time+6.28*o);
            float2 r = g + o - f;
            float d = dot(r, r);
            if (d<md) {
                md = d;
                m = n+g+o;
            } 
        }
    }
    return md;
}
//FBM 集合操作sum
float CausticVoronoi(float2 p,float time) {
    float v = 0.0;
    float a = 0.4;
    for (int i = 0;i<3;i++) {
        v+= wnoise(p,time)*a;
        p*=2.0;
        a*=0.5;
    }
    v = pow(v,2.)*5.;
    return v;
}
```

### **4.shader技巧总结 (重点)**
>如上所述，三种算法的结果的表现有类似的地方，这是一种技巧：  
1.重复多次类似操作 并收集结果 val1,val2,val3 ...  
2.将收集到的结果进行集合操作，如min,sum,avg等  
这样的操作的有很多如：  
.上面的算法  
.FBM(Fractional brownian motion)  
.Iterated Function Systems 如Mandelbrot Set,KaliSet  
.复杂的水的波的合成  
甚至泰勒公式从某种程度上来说也是类似的  
 
这个技巧非常重要，利用它，我们可以实现的效果就非常的多，如后面我们要实现的海洋，大风，云，山脉


### 5.算法思想来源
https://www.shadertoy.com/view/MdKXDm  
https://www.shadertoy.com/view/MdlXz8  


- [本教程配套blog ][1]
- [本教程配套项目源码 ][2]
- [教程中抽取的RayMarching框架][3]


  [1]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [2]: https://github.com/JiepengTan/FishManShaderTutorial
  [3]: https://github.com/JiepengTan/Unity-Raymarching-Framework
  [4]: https://www.shadertoy.com/view/Xsd3DB