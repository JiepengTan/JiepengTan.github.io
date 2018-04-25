---
layout: post
title:  "中级Shader教程13 Stars"
date:   2018-04-13 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial shader sky stars
mathjax: true
---
<p align="center"> <img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Stars/head.gif?raw=true" width="512"/></p>
<p align="center"></p>  


 


### 1.实现原理  
1.使用球坐标来进行空间划分  
2.对每个空间划分的grid产生hash  
3.根据hashID 定义星星的大小，闪烁周期，明暗程度等  
4.在grid绘制圆 用smoothstep 来控制圆的大小  


### 2.源码

1.单层FBM中不同的层之间移动速度随时间的偏移  

```c
// create by JiepengTan 2018-04-14 
// email: jiepengtan@gmail.com
Shader "FishManShaderTutorial/SDFBounceBall" {
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
#include "ShaderLibs/FullFramework3D.cginc"

            fixed SdBounceBalls(fixed3 pos){
                fixed SIZE = 2.;
                fixed2 gridSize = fixed2(SIZE,SIZE);
                fixed rv = Hash12( floor((pos.xz) / gridSize));
                pos.xz = OpRep(pos.xz,gridSize);
                fixed bollSize = 0.1;
                fixed bounceH = .5;
                return SdSphere(pos- fixed3(0.,(bollSize+bounceH+sin(_Time.y*3.14 + rv*6.24)*bounceH),0.),bollSize);
            }

            fixed2 Map( in fixed3 pos )
            {
                fixed2 res = fixed2( SdPlane(     pos), 1.0 )  ;
                res = OpU( res, fixed2( SdBounceBalls( pos),1.) );
                return res;
            }

            ENDCG
        }//end pass
    }//end SubShader
    FallBack Off
}
```


### 2.unity shader源码
shader源码可以在这里[下载][1]


  [1]: https://github.com/JiepengTan/FishManShaderTutorial