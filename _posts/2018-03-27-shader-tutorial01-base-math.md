---
layout: post
title:  "中级Shader教程01 基础函数"
date:   2018-03-27 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial mathmatic shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
 
TODO 
基础函数: Smoothstep Sin Clamp Pow Length Sqrt 
辅助函数: Saw WithIn Remap
图形: Circle Rect 





#图形
**1.绘制圆点**
```c
//假设uv 是 (-0.5,-0.5) 到(.5,0.5)
//blur 是模糊程度 值越大边缘越模糊 0.0完全不模糊 1.0为正常模糊 负数 反向
inline float DrawCircle(float2 uv,float blur){
	float val = 1.0-length(uv);
	val = smoothstep(0.5,0.500001+blur*0.5,val);
	return val;
}
```

下面是从左到右依次为 blur=-0.1   blur=0.0   blur=0.1  blur=1.0的图像：

<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/BaseMath/circle-01.jpg?raw=true" width="128">   <img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/BaseMath/circle00.jpg?raw=true" width="128">     <img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/BaseMath/circle01.jpg?raw=true" width="128">  <img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/BaseMath/circle10.jpg?raw=true" width="128">          


**随机值**
**.添加随机值**
```c
fixed2 Rand22(fixed2 co){
	fixed x = frac(sin(dot(co.xy ,fixed2(122.9898,783.233))) * 43758.5453);
	fixed y = frac(sin(dot(co.xy ,fixed2(457.6537,537.2793))) * 37573.5913);
	return fixed2(x,y);
}
```
fract(sin(x*bigVal1)*bigVal2)产生的数比较随机的原因是： 
sin(x*bigVal1) 会将x值的变化波动被放大，且随机，设这个值的变化为detX,则 frac(detX*bigVal2) 会让本来的小的波动再次放大，然后fract会让整体的数的变化受影响的因素变多，所以得到的了一个较为随机的值