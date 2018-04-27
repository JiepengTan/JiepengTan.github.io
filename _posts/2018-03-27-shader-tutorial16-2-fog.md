---
layout: post
title:  "中级Shader教程16.2 动态雾"
date:   2018-04-23 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial UI shader fog noise
mathjax: true
---
<p align="center"> <img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Fog/head.gif?raw=true" width="512"/></p>






### 1.前置知识链接    
1.Noise和FBM请参考[这篇文章中给出的链接][4]     
2.[ranymarching 框架][5]   

## 2.实现原理  
1.利用FBM来模拟空间雾的密度分布  
2.通过raymarch 距离根据密度从前到后多层颜色混合

这里的Noise 使用triNoise,其实PerlinNoise 也行，不过perlinNoise,需要的指令较多  

## 3.源码实现  
### **1.noise 模拟 fog 浓度**  
1.noise  
```c
float _tri(in float x){return abs(frac(x)-.5);}
float2 _tri2(in float2 p){return float2(_tri(p.x+_tri(p.y*2.)), _tri(p.y+_tri(p.x*2.)));}
float3 _tri3(in float3 p){return float3(_tri(p.z+_tri(p.y*1.)), _tri(p.z+_tri(p.x*1.)), _tri(p.y+_tri(p.x*1.)));}

//https://www.shadertoy.com/view/4ts3z2 
float TNoise(in float3 p, float time,float spd)
{
    float z=1.4;
    float rz = 0.;
    float3 bp = p;
    //类似FBM 效果 但是指令更少
    for (float i=0.; i<=3.; i++ )
    {
        float3 dg = _tri3(bp*2.);
        p += dg+time*spd;

        bp *= 1.8;
        z *= 1.5;
        p *= 1.2;
        
        rz+= (_tri(p.z+_tri(p.x+_tri(p.y))))/z;
        bp += 0.14;
    }
    return rz;
}
```

整个场景中Noise 分布 效果如下

<p align="center"> <img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Fog/noise.gif?raw=true" width="512"/></p>
<p align="center"></p>  

### **2.Fog实现**   
通过FBM 给整个场景中的noise 添加更多的变化，然后通过raymarch 从前王rayhit 的方向进行颜色混合来模拟fog 对于最终场景的影响。  
注意这里因为速度问题，我们采样的量不会太多，为了是雾气效果平滑的过渡，我们需要对采样点之间进行插值过渡  

```c

fixed3 Fog(in fixed3 bgCol, in fixed3 ro, in fixed3 rd, in fixed maxT,
                float3 fogCol,float3 spd,float2 heightRange)
{
    fixed d = .4;
    float3 col = bgCol;
    for(int i=0; i<7; i++)
    {
        fixed3  p = ro + rd*d;
        // add some movement at some dir
        p += spd * ftime;
        p.z += sin(p.x*.5);
        // get height desity 
        float hDen = (1.-smoothstep(heightRange.x,heightRange.y,p.y));
        // get final  density
        fixed den = TNoise(p*2.2/(d+20.),ftime, 0.2)* hDen;
        fixed3 col2 = fogCol *( den *0.5+0.5);
        col = lerp(col,col2,clamp(den*smoothstep(d-0.4,d+2.+d*.75,maxT),0.,1.) );
        d *= 1.5+0.3; 
        if (d>maxT)break;
    }
    return col;
}
```

### **3.全部源码**  

```c
// create by JiepengTan 2018-04-13  
// email: jiepengtan@gmail.com
Shader "FishManShaderTutorial/Fog" {
    Properties{
        _MainTex("Base (RGB)", 2D) = "white" {}
        _LoopNum ("_LoopNum", Vector) = (40.,128., 1, 1)
        _FogSpd ("_FogSpd", Vector) = (1.,0.,0.,0.5)
        _FogHighRange ("_FogHighRange", Vector) = (-5,10,0.,0.5)
        _FogCol ("_FogCol", COLOR) = (.025, .2, .125,0.)
    }
    SubShader{ 
        Pass {
            ZTest Always Cull Off ZWrite Off
            CGPROGRAM
            float4 _LoopNum = float4(40.,128.,0.,0.);
            float3 _FogSpd ;
            float2 _FogHighRange;
            fixed3 _FogCol;
             
#pragma vertex vert  
#pragma fragment frag  
#include "ShaderLibs/Framework3D.cginc" 
            #define ITR 100 
            #define FAR 50.
            fixed3 Normal(in fixed3 p){  
                return float3(0.,1.0,0.);
            }

            fixed RayCast(in fixed3 ro, in fixed3 rd) {
                if (rd.y>=0.0) {
                    return 100000;
                }
                float d = -(ro.y - 0.)/rd.y;
                d = min(100000.0, d);
                return d;
            }
            float4 ProcessRayMarch(float2 uv,float3 ro,float3 rd,inout float sceneDep,float4 sceneCol){ 
                fixed3 ligt = normalize( fixed3(.5, .05, -.2) );
                fixed3 ligt2 = normalize( fixed3(.5, -.1, -.2) );
    
                fixed rz = RayCast(ro,rd);
                fixed3 fogb = lerp(fixed3(.7,.8,.8  )*0.3, fixed3(1.,1.,.77)*.95, pow(dot(rd,ligt2)+1.2, 2.5)*.25);
                fogb *= clamp(rd.y*.5+.6, 0., 1.);
                fixed3 col = fogb;

                if ( rz < FAR ){
                    fixed3 pos = ro+rz*rd;
                    fixed3 nor= Normal( pos );
                    fixed dif = clamp( dot( nor, ligt ), 0.0, 1.0 );
                    fixed spe = pow(clamp( dot( reflect(rd,nor), ligt ), 0.0, 1.0 ),50.);
                    col = lerp(fixed3(0.1,0.2,1),fixed3(.3,.5,1),pos.y*.5)*0.2+.1;
                    col = col*dif + col*spe*.5 ;
                }
                 
                MergeRayMarchingIntoUnity(rz,col,sceneDep,sceneCol);  
            
                col = lerp(col, fogb, smoothstep(FAR-7.,FAR,rz)); 
                //then volumetric fog 
                col = Fog(col, ro, rd, rz,_FogCol,_FogSpd,_FogHighRange);
                //post
                col = pow(col,float3(0.8,0.8,0.8));
                sceneCol.xyz = col;
                return sceneCol; 
            }
            ENDCG
        }//end pass 
    }//end SubShader
    FallBack Off
}

```


- [本教程配套blog ][1]
- [本教程配套项目源码 ][2]
- [教程中抽取的RayMarching框架][3]

  [1]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [2]: https://github.com/JiepengTan/FishManShaderTutorial
  [3]: https://github.com/JiepengTan/Unity-Raymarching-Framework
  [4]: https://jiepengtan.github.io/2018/03/27/shader-tutorial01-base-math/
  [5]: https://jiepengtan.github.io/2018/04/22/shader-tutorial09-1-raymarch-framework/