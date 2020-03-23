---
layout: post
title:  "中级Shader教程11 Default渲染框架"
date:   2018-04-23 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial 3D_Raymarch shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---

###  **1.默认渲染框架**
文件Framework3D_DefaultRender.cginc说明：
该库封装了基本raymarching渲染，常用于测试SDF建模，
只需要自己定义Map函数(SDF描述),用于构建整个场景。







如果想要使用默认的渲染  
可以在shader中使用这几个宏即可    

```c
 #define DEFAULT_RENDER
 #define DEFAULT_MAT_COL
 #define DEFAULT_PROCESS_FRAG
```

如果需要需要进行自定义着色 重新定义  
```c
float3 MatCol(float matID,float3 pos,float3 nor);  
float2 Map( in float3 pos );  
float3 Render( in float3 ro, in float3 rd );  
```

这几个函数即可  

```c
//Framework3D_DefaultRender.cginc
float3 MatCol(float matID,float3 pos,float3 nor);
float2 Map( in float3 pos );
float3 Render( in float3 ro, in float3 rd );

#ifdef DEFAULT_RENDER
float3 Render( in float3 ro, in float3 rd )
{ 
    float3 col = float3(0.7, 0.9, 1.0) +rd.y*0.8;
    ...
}
#endif
#ifdef DEFAULT_MAT_COL
float3 MatCol(float matID,float3 pos,float3 nor)
{ 
    // material        
    float3 col = 0.45 + 0.35*sin( float3(0.05,0.08,0.10)*(matID-1.0) );
    if( matID<1.5 )
    {       
        float f = CheckersGradBox( 5.0*pos.xz );
        col = 0.3 + f*float3(0.1,0.1,0.1);
    }
    return col;
}
#endif

#ifdef DEFAULT_PROCESS_FRAG
float4 ProcessRayMarch(float2 uv,float3 ro,float3 rd,inout float sceneDep,float4 sceneCol)  {
        // render   
    float3 col = Render( ro, rd );
    // gamma
    col = pow( col, float3(0.4545,0.4545,0.4545) );
    sceneCol.xyz = col;  
    return sceneCol;
} 
#endif
```


## [**配套视频**][40]  
- [本教程配套blog ][1]
- [本教程配套项目源码 ][2]
- [教程中抽取的RayMarching框架][3]
- [个人blog ][4]

  [1]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [2]: https://github.com/JiepengTan/FishManShaderTutorial
  [40]:https://space.bilibili.com/308864667/channel/detail?cid=112754
  [3]: https://github.com/JiepengTan/Unity-Raymarching-Framework
  [4]: https://jiepengtan.github.io/
  