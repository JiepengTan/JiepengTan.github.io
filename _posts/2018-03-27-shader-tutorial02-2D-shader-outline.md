---
layout: post
title:  "中级Shader教程02 2Dshader总纲"
date:   2018-03-27 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial mathmatic shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---




### **1.概论**
### **shader中函数的基本理解**
 >1.length(uv) 降维效果 length(uv)将2D的转换为1D  
    2.atan(u,v) 获取角度 配合length 可以得到极坐标 
    3.smoothstep 获取较为平滑的过渡效果
    4.pow(f,n) 将曲线变化变得平滑或尖锐
    5.sin cos 周期函数，用于实现周期的效果(如来回移动，循环的移动等)
    6.hash 获取标志ID，常用于基于空间划分的效果的实现
    6.noise 同时具有随机性和连续性的函数 
    7.fbm 在基本函数的基础上，不断的叠加高频信息，丰富细节
### **2Dshader实现套路** 
>1.将整个UV进行空间划分 使用uv*scale
    2.基于每个划分后grid获取ID 使用floor
    3.给与每个grid 一个随机值 Hash22(ID) 
    4.基于随机值来实现各种效果
### **范例中包含新的特性有** 
####1. 2DSea
>. length
>. atan 极坐标
>. smoothstep
>. sin cos

####2. 2DSnow
>. hash
>. 空间划分

####3.2DFireParticle
>. hash
>. nois

####4.2DLava
>. fbm
----------

### **7.链接：**
- [本教程配套项目源码 ][1]
- [本人shadertoy地址 ][2]
- [第一时间更新blog地址][3]

  [1]: https://github.com/JiepengTan/FishManShaderTutorial
  [2]: https://www.shadertoy.com/user/FishMan
  [3]: https://jiepengtan.github.io/

