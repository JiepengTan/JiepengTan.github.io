---
layout: post
title:  "中级Shader教程06 2D火焰粒子"
date:   2018-03-27 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial fire particle grid shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---

 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/FireParticle/head.gif?raw=true" width="256"></p> 

### **0.本篇主要技术点有：**  
1. grid 空间划分  
2. 2D模拟3D中"分层"的概念  
3. 透视模拟
4. 基于grid的随机变化(旋转,位移,闪烁)
5. sin 周期变化(闪烁)




###  **1.实现**
#### 1.空间划分  
```c
float2 coord = uv*_GridSize;
```

 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/FireParticle/uvgrid01.jpg?raw=true" width="256"></p>

#### 2.交错grid  
```c
 if (abs(fmod(coord.y,2.0))<1.0) //让格子交错
        coord.x += 0.5;
```

 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/FireParticle/offset.jpg?raw=true" width="256"></p> 

#### 3.基于grid的随机化  
```c
float2 gridIndex = float2(floor(coord));
float rnd = Hash12(gridIndex);//根据ID 获取hash值
```
 
 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/FireParticle/rand.jpg?raw=true" width="256"></p> 

#### 4.整体位移  
```c
float2 coord = uv*_GridSize - float2(0.,yOffset);//整体沿y轴上升
float tempY = gridIndex.y + yOffset ;
float life = min(   10.0*(1.0-min((tempY)/(24.0-20.0*rnd)//生命值随机
                                    ,1.0)),1.0);

```

 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/FireParticle/moveup.gif?raw=true" width="256"></p> 

#### 5.加点闪烁  
```c
float sinval = sin(PI*1.*(0.3+0.7*rnd)*ftime+rnd*10.);//加点亮度的变化实现闪烁  随周期变化
float period = clamp(pow(pow(sinval,5.),5.),0.,1.);
float blink =(0.8+0.8*abs(period));
```

 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/FireParticle/blink.gif?raw=true" width="256"></p> 

#### 6.加点旋转  
```c
float2 rotate = float2(sin(deg),cos(deg));//单位圆旋转偏移
float radius =  0.5-size*0.2;
float2 cirOffset = radius*rotate;//
float2 part = frac(coord-cirOffset) - 0.5 ;//让格子自己旋转起来 位置变 
float len = length(part);
```

 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/FireParticle/rotate.gif?raw=true" width="256"></p> 

#### 6.最终效果 
 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/FireParticle/full.gif?raw=true" width="512"></p> 

整体代码：
```c
float3 _Color;
float _GridSize;
float _RotSpd;
float _YSpd;
float3 ProcessFrag(float2 uv){
    float3 acc = float3(0.0,0.0,0.0);

    float rotDeg = 3.*_RotSpd * ftime;
    float yOffset = 4.*_YSpd* ftime;

    float2 coord = uv*_GridSize - float2(0.,yOffset);//整体沿y轴上升
    if (abs(fmod(coord.y,2.0))<1.0) //让格子交错
        coord.x += 0.5;
    float2 gridIndex = float2(floor(coord));
    float rnd = Hash12(gridIndex);//根据ID 获取hash值
    // 弥补y轴上升的逆差 获取原来的y值 
    // 同时因为gridIndex = floor(coord) 的原因  会让tempY值在锁定固定的grid的同时越来越大;
    float tempY = gridIndex.y + yOffset ;
    float life = min(   10.0*(1.0-min((tempY)/(24.0-20.0*rnd)//生命值随机
                                        ,1.0)),1.0);
    if (life>0.0 ) {
        float size = 0.08*rnd;//让大小随机化
        float deg = 999.0*rnd*2.0*PI + rotDeg*(0.5+0.5*rnd);//添加旋转随机化
        float2 rotate = float2(sin(deg),cos(deg));//单位圆旋转偏移
        float radius =  0.5-size*0.2;
        float2 cirOffset = radius*rotate;//
        float2 part = frac(coord-cirOffset) - 0.5 ;//让格子自己旋转起来 位置变 方向不变
        float len = length(part);
        float sparksGray = max(0.0,1.0 -len/size);//画圆
        float sinval = sin(PI*1.*(0.3+0.7*rnd)*ftime+rnd*10.);//加点亮度的变化实现闪烁 
        float period = clamp(pow(pow(sinval,5.),5.),0.,1.);
        float blink =(0.8+0.8*abs(period));
        acc = life*sparksGray*_Color*blink;
    }
    return acc;
}
```



- [本教程配套项目源码 ][1]
- [本人shadertoy地址 ][2]
- [第一时间更新blog地址][3]

  [1]: https://github.com/JiepengTan/FishManShaderTutorial
  [2]: https://www.shadertoy.com/user/FishMan
  [3]: https://jiepengtan.github.io/ 
  [4]: https://jiepengtan.github.io/