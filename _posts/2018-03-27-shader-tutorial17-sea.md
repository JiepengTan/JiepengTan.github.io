---
layout: post
title:  "中级Shader教程17 海洋渲染"
date:   2018-04-23 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial theory shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Sea/head.gif?raw=true" width="512"></p> 





### 1.前置知识链接  
1.Noise和FBM请参考[这篇文章中给出的链接][4]   
2.[ranymarching 框架][5]    

### 2.海浪型状的构造
#### 1.基本形状构造
1.基本周期函数  
2.扩散方向  
3.Noise扰动  
#### 2.性能分析
1.还是和地形渲染一样，我们这里采用多种分辨率的地形函数  
2.性能关键  
.基本周期函数  
.Nolise函数的实现  


#### 3.实现  
#### 0.基本框架  

##### 1.基本raymarching    
```c
#define Waves(pos,NUM)\
    float2 uv = pos.xz;\
    ....//不同的实现


float2 TerrainL(float3 pos){ 
    Waves(pos,5.);
} 
float2 TerrainM(float3 pos){
    Waves(pos,9.);
} 
float2 TerrainH(float3 pos){
    Waves(pos,24.);
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

##### 2.水渲染  
海水渲染需要考虑的有diffuse,specular,reflect,refract,fresnel这集中不同的效应  
```c
//基本渲染
float3 RenderSea(float3 pos, float3 rd,float rz, float3 nor, float3 lightDir) {  
    float fresnel = clamp(1.0 - dot(nor,-rd), 0.0, 1.0);
    fresnel = pow(fresnel,3.0) * 0.65;

    float3 reflected = Sky(pos,reflect(rd,nor),lightDir);    
    float3 diff = pow(dot(nor,lightDir) * 0.4 + 0.6,3.);
    float3 refracted = _SeaBaseColor + diff * _SeaWaterColor * 0.12;
    float3 col = lerp(refracted,reflected,fresnel);

    float spec=  pow(max(dot(reflect(rd,nor),lightDir),0.0),60.) * 3.;
    col += float3(spec,spec,spec);

    return col;
}
```
##### 3.海天的处理
在海水和天空之间过渡
```c
float4 ProcessRayMarch(float2 uv,float3 ro,float3 rd,inout float sceneDep,float4 sceneCol){ 
    float rz = RaycastTerrain(ro,rd).x; 
    float3 pos = ro + rd *rz;
    float3 nor = NormalTerrian(pos,rz);
     
    // color
    float3 skyCol = Sky(pos,rd,_LightDir);
    float3 seaCol = RenderSea(pos,rd,rz,nor,_LightDir);
    //让海水和天空的过渡平和点
    float3 col = lerp(skyCol,seaCol,pow(smoothstep(0.0,-0.05,rd.y),0.3));
    col = pow( col, float3(0.4545,0.4545,0.4545) );
    sceneCol.xyz = col;
    return sceneCol; 
}

```

这里介绍两种方式来够着波浪：  
1.使用多个发射点(圆形)发射不同频率，振幅的波来合成最终效果    
2.使用多个放射方向(直线)发射不同频率，振幅的波来合成最终效果     

##### 1.圆形波
```c
#define Waves(pos,NUM)\
    float2 uv = pos.xz;\
    float w = 0.0,sw = 0.0;\
    float iter = 0.0, ww = 1.0;\
    uv += ftime * 0.5;\
    // 类FBM 波合成
    for(int i=0;i<NUM;i++){\
        w += ww * Wave(uv * 0.06 , float2(sin(iter), cos(iter)) * 10.0, 2.0 + iter * 0.08, 2.0 + iter * 3.0);\
        sw += ww;\
        ww = lerp(ww, 0.0115, 0.4);\
        iter += 2.39996;\
    }\
    return float2(pos.y- w / sw*_SeaWaveHeight,1.);\

    
float Wave(float2 uv, float2 emitter, float speed, float phase){ 
    //uv += Noise(uv);// 是否使用noise 来扭曲采样点
    float dst = distance(uv, emitter);//圆形扩散 极坐标
    return pow((0.5 + 0.5 * sin(dst * phase - ftime * speed)), 5.0);
}

```
两个sin波叠加的效果    
<p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Sea/Animation 25.gif?raw=true" width="256"></p> 

多个sin波叠加的效果  
<p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Sea/Animation 26.gif?raw=true" width="256"></p> 

最终效果  
<p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Sea/head2.gif?raw=true" width="512"></p> 


##### 1.方向波
```c
#define Waves(pos,_LOOP_NUM)\
    float2 uv = pos.xz;\
    //float2x2(0.8,0.6,-0.6,0.8) * 2.0
    float2x2 octave_m = float2x2(1.6,1.2,-1.2,1.6);\
    float freq = _SeaFreq;\
    float amp = _SeaWaveHeight;\
    float choppy = _SeaChoppy;\
    uv.x *= 0.75;\
    float d, h = 0.0;   \
    //类FBM效果  
    for(int i = 0; i < _LOOP_NUM; i++) {        \
        //让波浪相交
        d = Wave((uv+SEA_TIME)*freq,choppy);\
        d += Wave((uv-SEA_TIME)*freq,choppy);\
        h += d * amp;  \
        uv = mul(octave_m,uv); freq *= 1.9; amp *= 0.22;\
        choppy = lerp(choppy,1.0,0.2);\
    }\
    return float2(pos.y - h,1.0);

    
// sea
float Wave(float2 uv, float choppy) {
    uv += Noise(uv);   // 是否使用noise 来扭曲采样点
    float2 wv = 1.0-abs(sin(uv));//让波尖锐
    float2 swv = abs(cos(uv));   //方型
    wv = lerp(wv,swv,wv);
    return pow(1.0-pow(wv.x * wv.y,0.65),choppy);
}
```
两个方向波的效果  
<p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Sea/Animation 27.gif?raw=true" width="256"></p> 
 

多个方向波的效果  
<p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Sea/Animation 28.gif?raw=true" width="256"></p> 


 #### 2.随机化
 ```c
 float Wave(float2 uv, float choppy) {
    uv += Noise(uv);   // 是否使用noise 来扭曲采样点
    ...
 }
 ```
添加Noise来增加波浪的随机行为  
 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Sea/Animation 29.gif?raw=true" width="256"></p> 

最终效果    
 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Sea/head.gif?raw=true" width="512"></p> 



- [本教程配套blog ][1]
- [本教程配套项目源码 ][2]
- [教程中抽取的RayMarching框架][3]

  [1]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [2]: https://github.com/JiepengTan/FishManShaderTutorial
  [3]: https://github.com/JiepengTan/Unity-Raymarching-Framework
  [4]: https://jiepengtan.github.io/2018/03/27/shader-tutorial01-base-math/
  [5]: https://jiepengtan.github.io/2018/04/22/shader-tutorial09-1-raymarch-framework/

