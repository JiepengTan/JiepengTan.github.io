---
layout: post
title:  "中级Shader教程15 地形渲染"
date:   2018-04-23 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial mountain shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
<p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Mountain/head.gif?raw=true" width="512"></p> 





### 1.前置知识链接  
1.FBM请参考[这篇文章中给出的链接][4]  
2.[ranymarching 框架][5]  

### 2.抽取地形shader 通用函数
　　在处理类地形raymarching的过程中，为了优化，在raycast阶段会有一个大循环，为了性能，我们在这里采样的精度低一点，在计算法线的时候精度高一点，因为计算法线的时候，只有一次。  

所以我们分为三种不同精度的Map函数,同样为了减少重复代码，我们使用宏来展开  

```c
#define Terrain(pos,NUM)\
    float2 p = pos.xz*.9/_MaxTerrianH;\
    float a = 0.0;\
    float b = 0.491;\
    float2  d = float2(0.0,0.);\
    for( int i=0; i<NUM; i++ ){\
        float n = Noised(p).x;\
        a += b*n;\
        b *= 0.49;\
        p = p*2.01;\
    }\
    return float2(pos.y - _MaxTerrianH*a,1.);

float2 TerrainL(float3 pos){ 
    Terrain(pos,5.);
} 
float2 TerrainM(float3 pos){
    Terrain(pos,9.);
} 
float2 TerrainH(float3 pos){
    Terrain(pos,15.);
}  


float RaycastTerrain(float3 ro, float3 rd) { 
    _MRCRO_RAY_CAST(ro,rd,10000.,TerrainL);  
}
float3 NormalTerrian( in float3 pos, float rz ){
    _MACRO_CALC_NORMAL(pos,rz,TerrainH); 
}

float SoftShadow(in float3 ro, in float3 rd,float tmax){    
    _MACRO_SOFT_SHADOW(ro,rd,tmax,TerrainM);  
}  

```

### 3.地形渲染  
1.地形Map函数  
我们使用FBM来模拟地形的高程图  
```c
float2 MapTerrain(float3 pos){
    float2 p = pos.xz*.9/_MaxTerrianH;//调整地形的宽高比
    float a = 0.0;
    float b = 0.491;
    float2  d = float2(0.0,0.);
    for( int i=0; i<5.; i++ ){//基本fbm
        float n = Noised(p).x;
        a += b*n;
        b *= 0.49;
        p = p*2.01;
    }
    return float2(pos.y - _MaxTerrianH*a,1.);
}
```

2.地形着色  
　　这部分和传统的材质shader没什么区别，你需要考虑的有阴影，diffuse，specular,环境光，背光，以及雾效果。当然这里没有进行贴图处理，在unity中我们可以方便的添加纹理贴图。你只需添加相应的代码即可。  
在shadertoy中，如果没有纹理可用，这时候，就可以实用哦个procedures纹理，如fbm等生成。  

```c
 float3 RenderMountain(float3 pos, float3 rd,float rz, float3 nor, float3 lightDir) {  
    float3 col = float3(0.,0.,0.);
    //base color 
    col = float3(0.10,0.09,0.08);

    //lighting     
    float amb = clamp(0.5+0.5*nor.y,0.0,1.0);
    float dif = clamp( dot( lightDir, nor ), 0.0, 1.0 );
    float bac = clamp( 0.2 + 0.8*dot( normalize( float3(-lightDir.x, 0.0, lightDir.z ) ), nor ), 0.0, 1.0 );
    
    //shadow
    float sh = SoftShadow(pos+lightDir*_MaxTerrianH*0.08,lightDir,_MaxTerrianH*1.2);

    //brdf 
    float3 lin  = float3(0.0,0.0,0.0);
    lin += dif*float3(7.00,5.00,3.00)*float3( sh, sh*sh*0.5+0.5*sh, sh*sh*0.8+0.2*sh );
    lin += amb*float3(0.40,0.60,1.00)*1.2;
    lin += bac*float3(0.40,0.50,0.60);
    col *= lin;
    // fog
    float fo = 1.0-exp(-pow(0.1*rz/_MaxTerrianH,1.5));
    float3 fco = 0.65*float3(0.4,0.65,1.0);
    col = lerp( col, fco, fo );
    return col;
}

```

3.整合  
raycast检测碰撞点，如果没有碰撞到，就采样天空盒，有碰撞到，就进行响应的shading就好了。最后添加gamma矫正。  

```c
  float4 ProcessRayMarch(float2 uv,float3 ro,float3 rd,inout float sceneDep,float4 sceneCol){ 
    float tmax = 3000.;
    float rz = RaycastTerrain(ro,rd).x; 
    float3 pos = ro + rd *rz;
    float3 nor = NormalTerrian(pos,rz);
    // color 
    float3 col = float3(0.,0.,0.);
    if(rz >tmax ){ 
        col= Sky(pos,rd,_LightDir);
    }else{
        col = RenderMountain(pos,rd,rz,nor,_LightDir);
    } 
    //gamma
    col = pow( col, float3(0.4545,0.4545,0.4545) );
    sceneCol.xyz = col; 
    return sceneCol; 
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