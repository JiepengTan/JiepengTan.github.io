---
layout: post
title:  "中级Shader教程12 跳动的小球"
date:   2018-04-23 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial shader sky stars
mathjax: true
---
<p align="center"> <img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/BounceBall/head.gif?raw=true" width="512"/></p>





### 1.实现原理  
1.空间划分  
2.SDF

### 2.源码
简单代码如下：

```c
// create by JiepengTan 
// https://github.com/JiepengTan/FishManShaderTutorial2018-04-14 
// email: jiepengtan@gmail.com
Shader "FishManShaderTutorial/SDFBouncedBall" {
    Properties{
        _MainTex("Base (RGB)", 2D) = "white" {}
    }
    SubShader{
        Pass {
            ZTest Always Cull Off ZWrite Off
            CGPROGRAM

#pragma vertex vert   
#pragma fragment frag  

#define DEFAULT_MAT_COL
#define DEFAULT_PROCESS_FRAG
#define DEFAULT_RENDER
#include "ShaderLibs/Framework3D_DefaultRender.cginc"

            float SdBounceBalls(float3 pos){
                float SIZE = 2.;
                float2 gridSize = float2(SIZE,SIZE);
                //randval 让跳动随机化
                float rv = Hash12( floor((pos.xz) / gridSize));
                pos.xz = OpRep(pos.xz,gridSize);//空间划分
                float bollSize = 0.1;
                float bounceH = .5;
                //SDF 构建球体
                return SdSphere(pos- float3(0.,(bollSize+bounceH+sin(_Time.y*3.14 + rv*6.24)*bounceH),0.),bollSize);
            }

            float2 Map( in float3 pos )
            {
                float2 res = float2( SdPlane(     pos), 1.0 )  ;
                res = OpU( res, float2( SdBounceBalls( pos),1.) );
                return res;
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