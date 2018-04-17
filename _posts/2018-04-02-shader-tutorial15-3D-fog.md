---
layout: post
title:  "中级Shader教程13 3d Fog"
date:   2018-04-16 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial UI shader fog noise
mathjax: true
---
<p align="center"> <img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial3D/Fog/head.gif" width="512"/></p>
<p align="center"></p>  





## 1.实现原理  
1.利用FBM来模拟空间雾的密度分布  
2.通过raymarch 距离根据密度从前到后多层颜色混合

这里的Noise 使用triNoise,其实PerlinNoise 也行，不过perlinNoise,需要的指令较多  

## 2.源码实现  
### **1.noise 模拟 fog 浓度**  
1.noise  
```
float tri(in float x){return abs(frac(x)-.5);}
float2 tri2(in float2 p){return float2(tri(p.x+tri(p.y*2.)), tri(p.y+tri(p.x*2.)));}
float3 tri3(in float3 p){return float3(tri(p.z+tri(p.y*1.)), tri(p.z+tri(p.x*1.)), tri(p.y+tri(p.x*1.)));}

float tnoise(in float3 p, float time,float spd)
{
    float z=1.4;
    float rz = 0.;
    float3 bp = p;
    for (float i=0.; i<=3.; i++ )
    {
        float3 dg = tri3(bp*2.);
        p += dg+time*spd;

        bp *= 1.8;
        z *= 1.5;
        p *= 1.2;
        
        rz+= (tri(p.z+tri(p.x+tri(p.y))))/z;
        bp += 0.14;
    }
    return rz;
}
```

整个场景中Noise 分布 效果如下

<p align="center"> <img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial3D/Fog/noise.gif" width="512"/></p>
<p align="center"></p>  

### **2.FBM +Raymarch**   
通过FBM 给整个场景中的noise 添加更多的变化，然后通过raymarch 从前王rayhit 的方向进行颜色混合来模拟fog 对于最终场景的影响。  
注意这里因为速度问题，我们采样的量不会太多，为了是雾气效果平滑的过渡，我们需要对采样点之间进行插值过渡  

```c
fixed fogmap(in fixed3 p, in fixed d)
{
    p += _FogSpd.xyz * time;//添加雾气移动
    p.z += sin(p.x*.5);
    // 由3D noise 来模拟雾的浓度分布
    return tnoise(p*2.2/(d+20.),time, 0.2)*(1.-smoothstep(_FogHighRange.x,_FogHighRange.y,p.y));
}

fixed3 fog(in fixed3 col, in fixed3 ro, in fixed3 rd, in fixed mt)
{
    fixed d = .4;
    // 类似FBM 套路
    for(int i=0; i<7; i++)
    {
        fixed3  pos = ro + rd*d;
        fixed rz = fogmap(pos, d);
        fixed3 col2 = _FogCol *( rz *0.5+0.5);//混合雾的颜色
        float smoothLayer = smoothstep(d-0.4,d+2.+d*.75,mt);
        col = lerp(col,col2,clamp(rz*smoothLayer,0.,1.) );
        d *= 1.5+0.3;
        if (d>mt)break;
    }
    return col;
}
```

### **3.整合Unity& rayMarch**  
通过读取Unity场景中的deepth 贴图，获取场景的z值,通过比较unity的z值与raymarch的碰撞点的t值,就好比unity中的ztest 一样

```c
// create by JiepengTan 2018-04-13  email: jiepengtan@gmail.com
Shader "FishManShaderTutorial/Fog" {
    Properties{
        _MainTex("Base (RGB)", 2D) = "white" {}
        _LoopNum ("_LoopNum", Vector) = (40.,128., 1, 1)
        _FogSpd ("_FogSpd", Vector) = (1.,0.,0.,0.5)//雾移动速度
        _FogHighRange ("_FogHighRange", Vector) = (-5,10,0.,0.5)//雾的高度分布
        _FogCol ("_FogCol", COLOR) = (.025, .2, .125,0.)//雾颜色
        
    }
    SubShader{
        Pass {
            ZTest Always Cull Off ZWrite Off
            CGPROGRAM
            float4 _LoopNum = float4(40.,128.,0.,0.);
            float4 _FogSpd ;
            float4 _FogHighRange;
            fixed3 _FogCol;
            
#pragma vertex VertMergeRayMarch  
#pragma fragment FragMergeRayMarch  
#include "ShaderLibs/MergeRayMarch.cginc"
            #define ITR 100
            #define FAR 30.
            #define time _Time.y
            fixed fogmap(in fixed3 p, in fixed d)
            {
                p += _FogSpd.xyz * time;//添加雾气移动
                p.z += sin(p.x*.5);
                // 由3D noise 来模拟雾的浓度分布
                return tnoise(p*2.2/(d+20.),time, 0.2)*(1.-smoothstep(_FogHighRange.x,_FogHighRange.y,p.y));
            }
        
            fixed3 fog(in fixed3 col, in fixed3 ro, in fixed3 rd, in fixed mt)
            {
                fixed d = .4;
                // 类似FBM 套路
                for(int i=0; i<7; i++)
                {
                    fixed3  pos = ro + rd*d;
                    fixed rz = fogmap(pos, d);
                    fixed3 col2 = _FogCol *( rz *0.5+0.5);//混合雾的颜色
                    col = lerp(col,col2,clamp(rz*smoothstep(d-0.4,d+2.+d*.75,mt),0.,1.) );
                    d *= 1.5+0.3;
                    if (d>mt)break;
                }
                return col;
            }
            fixed3 normal(in fixed3 p)
            {  
                return float3(0.,1.0,0.);
            }
            //这里只是一个地面 你可以替换成你自己的场景
            fixed march(in fixed3 ro, in fixed3 rd)
            {
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
    
                fixed rz = march(ro,rd);
                //根据光照方向修改雾的颜色
                fixed3 fogb = lerp(fixed3(.7,.8,.8  )*0.3, fixed3(1.,1.,.77)*.95, pow(dot(rd,ligt2)+1.2, 2.5)*.25);
                fogb *= clamp(rd.y*.5+.6, 0., 1.);
                fixed3 col = fogb;
                
                if ( rz < FAR )
                {
                    //计算地面颜色
                    fixed3 pos = ro+rz*rd;
                    fixed3 nor= normal( pos );
                    fixed dif = clamp( dot( nor, ligt ), 0.0, 1.0 );
                    fixed spe = pow(clamp( dot( reflect(rd,nor), ligt ), 0.0, 1.0 ),50.);
                    col = lerp(fixed3(0.1,0.2,1),fixed3(.3,.5,1),pos.y*.5)*0.2+.1;
                    col = col*dif + col*spe*.5 ;
                }
                //混合Unity颜色
                if(rz>sceneDep){
                    col = sceneCol;
                    rz = sceneDep;
                }
                //添加雾效果
                col = lerp(col, fogb, smoothstep(FAR-7.,FAR,rz));
                //then volumetric fog
                col = fog(col, ro, rd, rz);
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


## 3.unity shader源码
shader源码可以在这里[下载][1]


  [1]: https://github.com/JiepengTan/FishManShaderTutorial