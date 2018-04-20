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
<img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/BaseMath/offset_uv.jpg" width="128"> <img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/BaseMath/grid_offset_uv.jpg" width="128"> <img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/BaseMath/grid_uv_circle.jpg" width="128">

#### **2.每个grid获取ID**
```c
fixed2 Rand22(fixed2 co){
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
fixed2 r = Rand22(floor(uv));
col = fixed3(r,0.0);
```
 <img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/BaseMath/grid_rand_val.jpg" width="128">
 


### **2.透视效果分析**

使用2D模拟3D需要考虑透视问题，如下图：
<img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/BaseMath/perspective.jpg" width="512">

总体有以下几点：

>1. 前景看到的东西会比较**大** 
>2. 前景看到的东西会比较**少** 
>3. 前景看到的物体会比较**清晰**(模拟现实)

