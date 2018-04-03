---
layout: post
title:  "中级Shader教程09 2D雪花"
date:   2018-03-27 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial snow grid shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
<img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/Snow/snow.gif" width="256">

Shader教程开篇，希望大家会喜欢。



# 1.技巧总结：

 - 2D模拟3D注意事项
 - 格子化空间划分 

# 2.基本理论：
#### 1.**透视效果分析**

使用2D模拟3D需要考虑透视问题，如下图：
<img src="https://jiepengtan.github.io/assets/img/blog/ShaderTutorial2D/Snow/perspective.jpg" width="512">

总体有以下几点：

>1. 前景看到的东西会比较**大** 
>2. 前景看到的东西会比较**少** 
>3. 前景看到的物体会比较**清晰**(模拟现实)


#### 2.**格子化空间划分**

>程序实现对多个物体进行属性修改或创建的时候，往往会用到for循环，但是在shader中，for循环是每个pixel都要执行的，效率很低，而且从另外的角度来看，一个屏幕有大量的pixel，这本身就是一种潜在的大循环。所以在2D shader中for循环是被类似"pixel划分整个屏幕"这种空间划分的技巧所代替。(ps：3D中是对整个世界空间进行格子划分来实现for循环)

# 3.分析：
>ps:后面()中的数字表示前面透视中的特点

将上述知识具体到观察下雪这各场景中：
>1. 雪花前面的大点(1)
>2. 雪花前面的少点(2)
>3. 雪花前面的亮点(3)

其他的特点有：
>4. 整体是向下降落，因为透视关系后面的下降速度慢点(2)
>5. 现实中的雪花的薄厚程度不一
>6. 雪花的空间分布比较随机
>7. 雪花的下落过程的可能会左右移动

# 4.代码实现：

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

<img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/BaseMath/circle-01.jpg" width="128">   <img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/BaseMath/circle00.jpg" width="128">     <img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/BaseMath/circle01.jpg" width="128">  <img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/BaseMath/circle10.jpg" width="128">          


**2.下降**

**3.空间划分**

**4.多层模拟**

**5.添加透视特点**


图片范例
<img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/Snow/grid01.jpg" width="256">
<img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/Snow/grid02.jpg" width="256">
<img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/Snow/grid03.jpg" width="256">
<img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial2D/Snow/snow.gif" width="256">

[教程的项目下载地址][1]

  [1]: https://github.com/JiepengTan/FishManShaderTutorial