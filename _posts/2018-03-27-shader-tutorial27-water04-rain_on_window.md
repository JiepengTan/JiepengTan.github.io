---
layout: post
title:  "中级Shader教程27 水专题04:三种窗前雨滴效果"
date:   2018-04-26 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial rain grid shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Rain/head.gif?raw=true" width="512"></p>

本篇主要技术点有：
> 栅格化屏幕空间
> 函数设计技巧:"凑"
> 2D旋转




# .代码实现：

### 0. 栅格化
```c
float aspectRatio = 4.0;//每一行雨滴的宽高比
float tileNum = 5;//平铺数量
uv *= fixed2(tileNum * aspectRatio,tileNum);//栅格化uv
uv = frac(uv);
uv -=0.5;
```
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Rain/rain01.jpg?raw=true" width="256"></p>

### 1. 绘制主要雨滴

```c
float r = length(uv);
r = smoothstep(0.2,0.1,r);
return float2(r,0.);
```

<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Rain/rain02.jpg?raw=true" width="256"></p>

```c
uv.y *= aspectRatio;
```
变形矫正

<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Rain/rain03.jpg?raw=true" width="256"></p>


### 2. 绘制尾迹

```c
//添加尾迹
float tailTileNum = 3.0;
float2 tailUV =uv *  float2(1.0,tailTileNum);
tailUV.y = frac(tailUV.y) - 0.5;
tailUV.x *= tailTileNum;
float rtail = length(tailUV);
rtail = smoothstep(0.2,0.1,rtail);
```

<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Rain/rain04.jpg?raw=true" width="256"></p>

### 3. 尾迹塑形
```c
//在雨滴上面总共有
float rtail = length(tailUV);
//尾迹塑形
rtail *= uv.y;//上面的y值大 使得雨滴形状变小
rtail = smoothstep(0.2,0.1,rtail);
//切除掉大雨滴下面的部分
rtail *= smoothstep(0.2,0.3,uv.y);//0.2以下的部分雨滴太大，切掉
```
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Rain/rain05.jpg?raw=true" width="256"></p>

### 4. 融合大雨滴和尾迹，并给雨滴添加模拟法线
```c
float2 allUV = float2(rtail*tailUV+r*uv);
```

<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Rain/rain06.jpg?raw=true" width="256"></p>

### 5. 把雨滴法线用于采样背景贴图
```c
	fixed4 finalColor = tex2D(_MainTex, uv + Rain(uv)*2.);
```
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Rain/rain07.jpg?raw=true" width="256"></p>
### 6. 让整个雨滴动起来
```c
//让雨滴沿y轴向下移动
float ySpd = 0.1;
uv.y += time * ySpd;
uv *= fixed2(tileNum * aspectRatio,tileNum);//栅格化uv
```

### 7. 让雨滴滚动起来更有节奏
```c
//这里是屏幕空间
uv.y += time * PI2 /period / tileNum *0.45* 0.55;//加点y轴移动
//other code

float period = 5;//second per circle
float t = time * PI2 /period;
//这里是格子空间
//此处uv值范围为(-0.5,0.5)
uv.y += sin(t+sin(t+sin(t)*0.55))*0.45;
uv.y *= aspectRatio;
```

我们想让雨滴的Y轴移动的行为像下图，这样会让雨滴先快速移动，然后好像停顿一样，然后继续的感觉

雨滴移动的函数为 
y = (sin(x+sin(x+sin(x)*0.55)))+x*0.55

<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/BaseMath/DesignFunction/func1.jpg?raw=true" width="256"></p>

如你所见，类似的函数有很多，只要能得到你想要的形状，怎么样都可以，如何“凑”出这些形状的函数，请参考我前面的博客的内容

<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/BaseMath/DesignFunction/func9.jpg?raw=true" width="256"></p>

而具体到代码中 
令 funcVal = (sin(x+sin(x+sin(x)*0.55))) 本身 范围为[-1,1.0]
而我们格子初始的uv的y值域为[-0.5，0.5]
我们的水滴是通过length来实现的 所以中心在dropCenter=0处，为了消除栅格化导致的明显的空间分割影响，让上下两个水滴拥有碰撞的可能是非常的必要的，在uv.y+= funcVal * 0.45后，dropCenter 的取值范围为[-0.5,0.5],即可以出现在格子的两端，从而在视觉上格子之间的水滴是有可能碰撞的
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Rain/rain08.jpg?raw=true" width="256"></p>

而代码中的
0.0618 = PI2 /period / tileNum *0.45* 0.55

效果如下：
只有大雨滴个之中的移动效果

<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Rain/AropAnim.gif?raw=true" width="256"></p>

加上贴图后

<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Rain/TexdropAnim.gif?raw=true" width="256"></p>

### 8. 加点基于格子的随机值
```c
fixed2 idRand = Rand22(floor(uv));
t += idRand.x * PI2;//添加Y随机值
/////
uv.x += (idRand.x-.5)*.6;//添加x轴随机偏移
```
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Rain/rain09.jpg?raw=true" width="256"></p>

### 9. 添加斜率的变化，模拟风的效果
```c
flaot DEG2RAD = 3.14159 /180;
flaot ratoteDeg = 20.0 * DEG2RAD;
float s = sin(ratoteDeg);
float c = cos(ratoteDeg);
float2x2 rot = float2x2(c, -s, s, c);
uv = mul(rot,uv);
```
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Rain/rain10.jpg?raw=true" width="256"></p>

### 10. 多加几层不同大小的雨滴
```c
rainUV += Rains(uv,152.12,moveSpd);
rainUV += Rains(uv*2.32, 25.23, moveSpd);
```

最终效果图

<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Rain/RainBase.gif?raw=true" width="256"></p>


本shader原型来自[BigWIngs][4]


- [本教程配套blog ][1]
- [本教程配套项目源码 ][2]
- [教程中抽取的RayMarching框架][3]


  [1]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [2]: https://github.com/JiepengTan/FishManShaderTutorial
  [3]: https://github.com/JiepengTan/Unity-Raymarching-Framework
  [4]: https://www.shadertoy.com/view/MdfBRX