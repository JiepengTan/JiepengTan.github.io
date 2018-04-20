---
layout: post
title:  "中级Shader教程05 2D雪花"
date:   2018-03-27 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial snow grid shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
 <p align="center">
<img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/Snow/head.gif" width="512"></p> 




###  **1.原理：**

在[前面][4]中介绍了2D模拟3D中的一些小技巧，这里用到了透视以及空间划分的概念，这里将不再重复：

###  **2.分析：**
>ps:后面()中的数字表示前面透视中的特点
根据 中级Shader教程02 基本图形2D中介绍了相应的一些概念
将上述知识具体到观察下雪这各场景中：
>1. 雪花前面的大点(1)
>2. 雪花前面的少点(2)
>3. 雪花前面的亮点(3)

其他的特点有：
>4. 整体是向下降落，因为透视关系后面的下降速度慢点(2)
>5. 现实中的雪花的薄厚程度不一
>6. 雪花的空间分布比较随机
>7. 雪花的下落过程的可能会左右移动

###  **3.代码实现：**


#### **1.空间划分**

```c
uv *=10;//将uv放大后frac
uv = frac(uv);
uv-=0.5；
```

<img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/Snow/grid00.jpg" width="256">

#### **2.添加随机值**
```c
fixed2 Rand22(fixed2 co){
	fixed x = frac(sin(dot(co.xy ,fixed2(122.9898,783.233))) * 43758.5453);
	fixed y = frac(sin(dot(co.xy ,fixed2(457.6537,537.2793))) * 37573.5913);
	return fixed2(x,y);
}
fixed2 r = Rand22(floor(uv));
col = fixed3(r,0.0);
```
<img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/Snow/grid01.jpg" width="256">

#### **3.uv偏移**
```c
float2 rgrid = Rand22(floor(uv));//0~1.0
uv = frac(uv);
uv -= (rgrid*2.0-1.0) * 0.35;
uv -=0.5;
```
<img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/Snow/grid02.jpg" width="256">

#### **4.绘制基本图形**
```c
float r = length(uv);
```
<img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/Snow/grid03.jpg" width="256">
```c
float r = length(uv);
float circleSize = 0.3;
float val = smoothstep(circleSize,-circleSize,r);
float3 col = float3(val,val,val)* rgrid.x ;
```
<img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/Snow/grid04.jpg" width="256">

#### **5.添加不同的layer**
针对不同的layer 需要调节的参数有  
1.格子数量  
2.y轴移动速度  
3.x轴移动速度  
4.格子的随机值  

```c
#define SIZE_RATE 0.1
#define YSPEED 0.5
#define XSPEED 0.2
#define LAYERS 10
float Rand11(float x){
	return frac(sin(x*157.1147) * 43751.1353);
}
fixed2 Rand22(fixed2 co){
	fixed x = frac(sin(dot(co.xy ,fixed2(1232.9898,7183.233))) * 43758.5453);
	fixed y = frac(sin(dot(co.xy ,fixed2(4577.6537,5337.2793))) * 37573.5913);
	return fixed2(x,y);
}
float3 SnowSingleLayer(float2 uv,float layer){
	float time = _Time.y;
	fixed3 acc = fixed3(0.0,0.0,0.0);
	uv = uv * (2.0+layer);//透视视野变大效果
    float xOffset = uv.y * (((Rand11(layer)*2-1.)*0.5+1.)*XSPEED);//增加x轴移动
    float yOffset = (YSPEED*time);//y轴下落过程
	uv += fixed2(xOffset,yOffset);
	float2 rgrid = Rand22(floor(uv)+(31.1759*layer));
	uv = frac(uv);
	uv -= (rgrid*2-1.0) * 0.35;
	uv -=0.5;
	float r = length(uv);
	float circleSize = 0.05*(1.0+0.3*sin(time*SIZE_RATE));//让大小变化点
	float val = smoothstep(circleSize,-circleSize,r);
	float3 col = float3(val,val,val)* rgrid.x ;
	return col;
}
float3 Snow(float2 uv){
	float3 acc = float3(0,0,0);
	for (fixed i=0.;i<LAYERS;i++) {
		acc += SnowSingleLayer(uv,i); 
	}
	return acc;
}
fixed4 ProcessFrag(v2f input)  {
	return float4(Snow(input.uv),1.0);
}
```



- [本教程配套项目源码 ][1]
- [本人shadertoy地址 ][2]
- [第一时间更新blog地址][3]

  [1]: https://github.com/JiepengTan/FishManShaderTutorial
  [2]: https://www.shadertoy.com/user/FishMan
  [3]: https://jiepengtan.github.io/ 
  [4]: https://jiepengtan.github.io/