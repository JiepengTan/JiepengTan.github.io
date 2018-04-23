---
layout: post
title:  "中级Shader教程09 3D Raymarch框架"
date:   2018-04-22 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial 3D_Raymarch shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
终于，我们暂时结束了2D，进入了令人兴奋的3D!! 
在2D的屏幕中，绘制3D场景----升维进化！相信我，当你搞定了3D,再回头看2Dshader,你会想起一句广告词，so easy!妈妈再也不用担心我们的shader了.
 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/RayMarchingFramework/head.gif?raw=true" width="512"></p> 




### **1.3D Raymarch框架** 
>1. 获得相机位置ro
2. 根据相机位置和朝向,计算当前像素所发出的射线ray的方向rd(ray dir)
3. 求交ray和场景的碰撞点p(两种方式)
>3.1 直接算式求解(比如射线到一个简单的圆的交点)
 3.2 使用raymarching方式即一步步的递进ray,直到ray碰到场景，或达到ray的最大距离。
4. 求得p处的法线和材质信息
5. 根据4得到的信息求的p处的颜色


举个Raymarching方式例子：
```c
// create by JiepengTan 
// date:2018-04-12 
// email: jiepengtan@gmail.com
Shader "FishManShaderTutorial/RayMarchSimpleScene"{
    Properties {
        _MainTex ("Texture", 2D) = "white" {}
    }
    SubShader{
        Tags { "RenderType"="Opaque" }
        LOD 100
        Pass {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"
            //#include "ShaderLibs/Noise.cginc"
            struct appdata{
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f {
                float2 uv : TEXCOORD0;
                float4 vertex : SV_POSITION;
            };
            
            sampler2D _MainTex;
            float4 _MainTex_ST;

            v2f vert (appdata v){
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                return o;
            }

            #define SPHERE_ID (1.0)
            #define FLOOR_ID (2.0)
            #define lightDir (normalize(float3(5.,3.0,-1.0)))


            float MapSphere(float3 pos){
                // center at float3(0.,0.,0.);
                float radius = 0.5;
                float3 centerPos = float3(0.,1.0+ sin(_Time.y*1.)*0.5,0.);
                return length(pos-centerPos) - radius;
            }
            float MapFloor(float3 pos ){
                float3 n= float3(0.,1.,0.);
                float3 d = 0;
                return dot(n,pos)-d;
            }
            float2 Map(float3 pos){
                float dist2Sphere = MapSphere(pos);// ID 1
                float dist2Plane = MapFloor(pos); // ID 2
                if(dist2Plane < dist2Sphere) {
                    return float2(dist2Plane,FLOOR_ID);
                }else{
                    return float2(dist2Sphere,SPHERE_ID);
                }
            }


            #define MARCH_NUM 256 //最多光线检测次数
            float2 RayCast(float3 ro,float3 rd){
                float tmin = 0.1;
                float tmax = 20.0;
               
                float t = tmin;
                float2 res = float2(0.,-1.0);
                for( int i=0; i<MARCH_NUM; i++ )
                {
                    float precis = 0.0005;
                    float3 pos = ro+rd*t;
                    res = Map(pos);
                    if( res.x<precis || t > tmax ) break;
                    t += 0.5*res.x;// 加速检测速度 这里可以有不同的策略
                }
                if( t>tmax ) return float2(t,-1.0);
                return float2( t, res.y );
            }
            float SoftShadow(float3 ro, float3 rd )
            {
                float res = 1.0;
                float t = 0.001;
                for( int i=0; i<80; i++ )
                {
                    float3  p = ro + t*rd;
                    float h = Map(p);
                    res = min( res, 16.0*h/t );
                    t += h;
                    if( res<0.001 ||p.y>(200.0) ) break;
                }
                return clamp( res, 0.0, 1.0 );
            }

            float3 ShadingShpere(float3 rd,float3 pos, float3 n,float3 sd){
                float3 col = float3(1.,0.,0.);
                float diff = clamp(dot(n,lightDir),0.,1.);
                float bklig = clamp(dot(n,-lightDir),0.,1.)*0.05;//加点背光
                return col *(diff+bklig);
            }
            float3 ShadingFloor(float3 rd,float3 pos, float3 n,float3 sd ){
                float3 col = float3(0.,1.,0.);
                float diff = clamp(dot(n,lightDir),0.,1.);
                return col *diff*sd;
            }
            float3 ShadingBG(float3 rd,float3 pos, float3 n ){
                float val = pow(rd.y,2.0);
                float3 bCol =float3(0.,0.,0.);
                float3 uCol =float3(0.1,0.2,0.9);
                return lerp(bCol,uCol,val);
            }
            float3 Shading(float3 rd,float3 pos, float3 n ,float matID){
                float sd = SoftShadow(pos,lightDir);
                if(matID >= (FLOOR_ID-0.5)){
                    return ShadingFloor(rd,pos,n,sd);
                }else{
                    return ShadingShpere(rd,pos,n,sd);
                }
            }

            float3 Normal(float3 pos, float t){
                float val = 0.0001 * t*t;
                float3 eps = float3(val,0.,0.);
                float3 nor = float3(
                    Map(pos+eps.xyy).x - Map(pos-eps.xyy).x,
                    Map(pos+eps.yxy).x - Map(pos-eps.yxy).x,
                    Map(pos+eps.yyx).x - Map(pos-eps.yyx).x );
                return normalize(nor);
            }
            void SetCamera(float2 uv,out float3 ro, out float3 rd){
                //步骤1 获得相机位置ro
                ro = float3(0.,2.,-5.0);//获取相机的位置 
                float3 ta = float3(0.,0.5,0.);//获取目标位置
                float3 forward = normalize( ta - ro);//计算 forward 方向
                float3 left = normalize(cross( float3(0.0,1.0,0.0), forward ));//计算 left 方向
                float3 up = normalize(cross(forward,left));////计算 up 方向
                const float zoom = 1.;

                //步骤2 获得射线朝向
                rd = normalize( uv.x*left + uv.y*up + zoom*forward );
            }
            fixed4 frag (v2f i) : SV_Target
            {
                // map uv into [-0.5,0.5]
                float2 uv = (i.uv-0.5) * float2(_ScreenParams.x/_ScreenParams.y,1.0);
                float3 ro,rd;
                //步骤1 步骤2
                SetCamera(uv,ro,rd);
                //步骤3 求交ray和场景的碰撞点p 
                float2 ret = RayCast(ro,rd);
                float3 pos = ro+ret.x*rd;
                
                //步骤4 计算碰撞点的法线信息    
                float3 nor= Normal( pos, ret.x );
                
                //步骤5 使用步骤4获得的信息计算当前像素的的颜色值
                float3 col = Shading(rd, pos,nor,ret.y);
                if(ret.y < -0.5){
                    col = ShadingBG(rd,pos,nor);
                }
                return float4(col,1.0);
            }
 
            
            ENDCG 
        }//end pass
    }//end SubShader
    FallBack Off
}

              
```


###  **2.公用框架抽取**
#### **2.1.抽取SetCamera函数**
>和shadertoy不同的是，unity 为我们提供好了一个非常棒的场景编辑器，为了更加直观的操作和观察我们的shader场景,我们可以直接将unity中相机的信息穿进去。来初始化ro,rd。

#### **2.2.融合Unity场景和RayMarching场景**
>光栅化渲染的优点是非常高效的渲染任意形态的多边形，而raymarching的或者说是SDF的优点是非常擅长处理规则，公式化的场景。为了利用这两种不同的渲染方式的优点。我们希望能够将Unity渲染的场景和raymarching渲染的场景整合在一起。
这个整合的一个切入点是，目前光栅化硬件的处理方式是通过zbuffer的方式来实现多层物体的正确前后排序。整个场景通过投影到近裁剪面的方式来渲染。最终zBuffer中保存的就是在目标像素发出的射线所碰撞的离相机最近的不透明的点到相机的向量投影到相机forward轴的长度。  
从另外一种角度来看，光栅是一宗raymarching的逆形式。(一种是ray朝向相机，一种是ray远离相机)  
从相机的参数中我们可以获取rd相关信息,从zbuffer获取的值中我们可以计算得到射线到碰撞点的距离uz(unity z)。结合raymarching 中计算得到的碰撞点到相机的距离rz（ray z）,类似光栅中的ztest,我们比较uz和rz 选取较小的z值对于的shading值。

所以本教程使用的一种实现方式：
**1.获取rd**
通过相机的参数，计算相机近裁减面的四个角到相机的射线，(渲染的是一个四边形)然后通过顶点shader中采样，利用光栅化的过程，硬件加速插值来得到射线rd。

**2.计算rz**
从unity中获取深度贴图，并采样得到zVal,然后通过投影逆操作，计算得到rz
```c
float depth = LinearEyeDepth(SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv));
float rz = depth * length(i.interpolatedRay.xyz);//
```

**3.获取sceneColor**
从unity中获取ColorBuffer信息(使用 OnRenderImage+RenderTexture)，
```c
float4 ProcessRayMarch(float2 uv,float3 ro,float3 rd,inout float sceneDep,float4 sceneCol);
float4 frag(v2f i) : SV_Target{
    float depth = LinearEyeDepth(SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv_depth));
    float rz = depth * length(i.interpolatedRay.xyz);//
    fixed4 sceneCol = tex2D(_MainTex, i.uv);//获取Unity渲染结果
    float2 uv = i.uv * float2(_ScreenParams.x/_ScreenParams.y,1.0);
    fixed3 ro = _WorldSpaceCameraPos;
    fixed3 rd = normalize(i.interpolatedRay.xyz);//注意需要normalize
    return ProcessRayMarch(uv,ro,rd,rz,sceneCol);
}
```

将其封装后在Framework3D.cginc中，配合宏展开，我们将不再需要重写这段代码
```c
#pragma vertex vert   
#pragma fragment frag  
#include "ShaderLibs/Framework3D.cginc"
float4 ProcessRayMarch(float2 uv,float3 ro,float3 rd,inout float sceneDep,float4 sceneCol)  {
 ...//你自己的渲染代码
}
```

**4.比较zVal,选择最终需要渲染结果**  
有两种可能，一种是我们是将Unity渲染的场景作为主场景，RayMarching只是当成是一种PostProcess shader，这时候,我们不将RayMarching 的背景色写入ColorBuffer，使用MergeRayMarchingIntoUnity,例如[Fog场景][4]  
第二种是raymarching 是主场景，unity只是作为点缀。这时候，unity中的天空盒信息就不应该写入ColorBuffer,应该使用MergeUnityIntoRayMarching。例如[Lake in Highland][5]
```c
//RayMarching is the main world, ignore unity sky box
void MergeUnityIntoRayMarching(inout float rz,inout float3 rCol, float unityDep,float4 unityCol){
    if(rz>unityDep && unityDep<_ProjectionParams.z-1.){// unity camera far plane 
        rCol = unityCol.xyz;
        rz = unityDep;
    }
}

//Unity scene is the main world, ignore RayMarching sky box
void MergeRayMarchingIntoUnity(inout float rz,inout float3 rCol, float unityDep,float4 unityCol){
    if(rz>unityDep ){
        rCol = unityCol.xyz;
        rz = unityDep;
    }
}       
```

###  **3.另外一种3D渲染实现**
在这个框架的基础上，我们实现上面所提到的使用公式的方式计算ray到场景的碰撞点的方式。
省去了RayCast部分,整个渲染代码非常的短.（当然也因为场景简单，以及shading过程简单）
```c
Shader "FishManShaderTutorial/RaymarchMergeExample" {
    Properties{
        _MainTex("Base (RGB)", 2D) = "white" {}
    }
    SubShader{
        Pass {
            ZTest Always Cull Off ZWrite Off
            CGPROGRAM

#pragma vertex vert   
#pragma fragment frag  
#include "ShaderLibs/Framework3D.cginc"

            float4 ProcessRayMarch(float2 uv,float3 ro,float3 rd,inout float sceneDep,float4 sceneCol)  {
                float3 col = float3(0.,0.,0.);
                float3 n = float3(0.,1.0,0.);
                float t = sceneDep + 10;
                float occ = 0.;
                float3 sc =  float3(0.,1.0+0.5*sin(ftime*PI2),0.);
                float3 sr = 0.5;
                float3 ce = sc - ro;
                float b = dot( rd, ce );
                float tt = sr*sr - (dot( ce, ce )- b*b );
                if( tt > 0.0 ){
                    t = b - sqrt(tt);
                    float3 p = ro+t*rd;
                    col = 0.5+0.5*cos(2.*PI*(float3(1.,1.,1.)*p.y*0.2+float3(0.,0.33,0.67)));
                }
                MergeRayMarchingIntoUnity(t,col, sceneDep,sceneCol);
                return float4(col,1.0);
            } 
            ENDCG
        }//end pass
    }//end SubShader
    FallBack Off
}
```

效果:  

<p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/Show/MergeRaymarchExample.gif?raw=true" width="660"></p>  


- [本教程配套项目源码 ][1]
- [本人shadertoy地址 ][2]
- [第一时间更新blog地址][3]


  [1]: https://github.com/JiepengTan/FishManShaderTutorial
  [2]: https://www.shadertoy.com/user/FishMan
  [3]: https://jiepengtan.github.io/
  [4]: https://github.com/JiepengTan/FishManShaderTutorial/tree/master/Assets/FishManShaderTutorial/Scene/Fog.unity
  [5]: https://github.com/JiepengTan/FishManShaderTutorial/tree/master/Assets/FishManShaderTutorial/Scene/HighlandLake.unity