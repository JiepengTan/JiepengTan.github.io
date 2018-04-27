---
layout: post
title:  "中级Shader教程18 水渲染"
date:   2018-04-23 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial shader sky water lake mountain 
mathjax: true
---
 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Lake/head.gif?raw=true" width="512"></p>   






### 1.前置知识链接    
1.Noise和FBM请参考[这篇文章中给出的链接][4]     
2.[ranymarching 框架][5]      

### 2.分析  
#### 1.基本形状构造  
这里利用了Noise本身随机的同时也连续的特性来模拟水面的变动  
1.使用FBM来实现水面的基本形状  
2.水的动态是通过在Noise中使用time作为一个维度，来实现水的扰动  

#### 2.着色  
1.水渲染需要考虑的有diffuse,specular,reflect,refract,fresnel这几种不同的效应   

### 3.实现    
#### 1.波浪  
1.使用FBM 来模拟水的波动 (利用的是其随机性和连续性)    
2.在Noise中使用time作为其中的一维度来模拟水的动态    

```c
float WaterMap( fixed3 pos ) {
    return FBM( fixed3( pos.xz, ftime )) * 1;
}

float3 WaterNormal(float3 pos,float rz){
    //注意这里的小技巧 法线随着距离而变化幅度小的，
    //不会造成因ray采样少而导致的噪音波动
    float EPSILON = 0.003*rz*rz;
    float3 dx = float3( EPSILON, 0.,0. );
    float3 dz = float3( 0.,0., EPSILON );
        
    float3  normal = float3( 0., 1., 0. );
    float bumpfactor = 0.4 * pow(1.-clamp((rz)/1000.,0.,1.),6.);//根据距离所见减少Bump幅度
    
    normal.x = -bumpfactor * (WaterMap(pos + dx) - WaterMap(pos-dx) ) / (2. * EPSILON);
    normal.z = -bumpfactor * (WaterMap(pos + dz) - WaterMap(pos-dz) ) / (2. * EPSILON);
    return normalize( normal ); 
}
```

#### 1.折射反射效果模拟  

##### 1.反射
1.反射是根据入射rd以及法线nor，计算反射rdrfl  
2.通过反射rdrfl和入射点p 重新发射一条新的ray 进行raymarching  

##### 2.折射
1.在进行raymarching 的过程中先忽略水面的检测这样可以直接获得碰撞点p  
2.计算p到水面的距离 来融合水的颜色和背景色，模拟折射效果  

这里使用了取巧的方式只是改变了p的法线来模拟水面的扰动，正确的应该根据水面法线重新raymarching,


```c
  float4 ProcessRayMarch(float2 uv,float3 ro,float3 rd,inout float sceneDep,float4 sceneCol){  
    float maxT = 10000;
    float minT = 0.1;
    float3 col  = float3 (0.,0.,0.);
    float waterT = maxT;
    //直接模拟水平面  求交ray和平面的交点
    if(rd.y <-0.01){
        float t = -(ro.y - waterHeight)/rd.y;
        waterT = min(waterT,t);
    }
    float sundot = clamp(dot(rd,_LightDir),0.0,1.0);
    float rz = RaycastTerrain(ro,rd);
    float firstInsertRZ = min(rz,waterT);
    float fresnel = 0;
    float3 refractCol = float3(0.,0.,0.);
    bool reflected = false;
    // hit the water
    if(rz >= waterT && rd.y < -0.01){
        float3 waterPos = ro + rd * waterT; 
        float3 nor = WaterNormal(waterPos,waterT);
        float ndotr = dot(nor,-rd);
        fresnel = pow(1.0-abs(ndotr),6.);//
        float3 diff = pow(dot(nor,_LightDir) * 0.4 + 0.6,3.);
        // 基本水颜色 
        float3 waterCol = _BaseWaterColor + diff * _LightWaterColor * 0.12; 
        // 根据pos距离水面的高度调整透明程度
        float transPer = pow(1.0-clamp( rz - waterT,0,waterTranDeep)/waterTranDeep,3.);
        // 获取折射后颜色
        float3 bgCol = RenderMountain(ro,rd + nor* clamp(1.-dot(rd,-nor),0.,1.),rz);
        refractCol = lerp(waterCol,bgCol,transPer);
        //计算发射ro,rd，重新发射ray 进行 raymarching
        ro = waterPos;
        rd = reflect( rd, nor);
        rz = RaycastTerrain(ro,rd);
        reflected = true;
        col = refractCol; 
    }
    //渲染背景
    if(rz >= maxT){
        col = Sky( ro, rd,_LightDir);
    }else{
        col = RenderMountain(ro,rd,rz);
    }
    // 融合折射和反射效果
    if( reflected == true ) {
        col = lerp(refractCol,col,fresnel);
        float spec=  pow(max(dot(rd,_LightDir),0.0),128.) * 3.;
        col += float3(spec,spec,spec);
    }
    
    MergeUnityIntoRayMarching(firstInsertRZ,col,sceneDep,sceneCol); 
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