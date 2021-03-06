---
layout: post
title:  "中级Shader教程03 2Dshader框架"
date:   2018-03-27 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial theory shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
//





### **0.2Dshader设计流程**
>0.确定shader效果  
1.空间划分  
2.透视模拟  


### **1.空间划分**

>程序实现对多个物体进行属性修改或创建的时候，往往会用到for循环，但是在shader中，for循环是每个pixel都要执行的，效率很低，而且从另外的角度来看，一个屏幕有大量的pixel，这本身就是一种潜在的大循环。所以在2D shader中for循环是被类似"pixel划分整个屏幕"这种空间划分的技巧所代替。(ps：3D中是对整个世界空间进行格子划分来实现for循环)

举个例子：
在空间中绘制不同颜色的圆形格子
#### **1.空间划分**

```c
uv *=20;//将uv放大后frac
uv = frac(uv);
```

如下：从左到右一次为原始uv，grid后的uv,以及根据grid后的uv绘制的圆  

<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/BaseMath/offset_uv.jpg?raw=true" width="128"> <img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/BaseMath/grid_offset_uv.jpg?raw=true" width="128"> <img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/BaseMath/grid_uv_circle.jpg?raw=true" width="128">

#### **2.每个grid获取ID**
```c
fixed2 Hash22(fixed2 co){
    fixed x = frac(sin(dot(co.xy ,fixed2(122.9898,783.233))) * 43758.5453);
    fixed y = frac(sin(dot(co.xy ,fixed2(457.6537,537.2793))) * 37573.5913);
    return fixed2(x,y);
}
```
fract(sin(x*bigVal1)*bigVal2)产生的数比较随机的原因是： 
sin(x*bigVal1) 会将x值的变化波动被放大，且随机，设这个值的变化为detX,则 frac(detX*bigVal2) 会让本来的小的波动再次放大，然后fract会让整体的数的变化受影响的因素变多，所以得到的了一个较为随机的值

#### **3.添加随机值**
让每个栅格拥有不同的随机值
```c
fixed2 r = Hash22(floor(uv));
col = fixed3(r,0.0);
```

 <p align="center">
 <img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/BaseMath/grid_rand_val.jpg?raw=true" width="256"></p>


### **2.透视效果分析**

使用2D模拟3D需要考虑透视问题，如下图：
 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/BaseMath/perspective.jpg?raw=true" width="512"></p>

总体有以下几点：

>1. 前景看到的东西会比较**大** 
>2. 前景看到的东西会比较**少** 
>3. 前景看到的物体会比较**清晰**(模拟现实)


### **3.范例目录**
接下来的几个粒子都是使用这个套路实现。下面是几个场景中覆盖的知识点：  
#### [1.2DSea][7]
.length  
.atan 极坐标  
.smoothstep  
.sin cos

#### [2.2DSnow][8]  
.hash  
.空间划分  

#### [3.2DFireParticle ][9] 
.hash  
.nois

#### [4.2DLava ][10] 
.fbm  

----------

### **7.链接：**
## [**配套视频**][40]  
- [本教程配套blog ][1]
- [本教程配套项目源码 ][2]
- [教程中抽取的RayMarching框架][3]

  [1]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [2]: https://github.com/JiepengTan/FishManShaderTutorial
  [40]:https://space.bilibili.com/308864667/channel/detail?cid=112754
  [3]: https://github.com/JiepengTan/Unity-Raymarching-Framework
  [4]: https://jiepengtan.github.io/2018/03/27/shader-tutorial01-base-math/
  [5]: https://jiepengtan.github.io/2018/03/27/shader-tutorial02-shader-skills/
  [6]: https://jiepengtan.github.io/2018/03/27/shader-tutorial03-2D-shader-framework/
  [7]: https://jiepengtan.github.io/2018/03/27/shader-tutorial04-2D-sea/
  [8]: https://jiepengtan.github.io/2018/03/27/shader-tutorial05-2D-snow/
  [9]: https://jiepengtan.github.io/2018/03/27/shader-tutorial06-2D-fire-particle/
  [10]: https://jiepengtan.github.io/2018/03/27/shader-tutorial07-2D-lava/
  [11]: https://jiepengtan.github.io/2018/04/22/shader-tutorial09-1-raymarch-framework/
  [12]: https://jiepengtan.github.io/2018/04/23/shader-tutorial10-SDF/

