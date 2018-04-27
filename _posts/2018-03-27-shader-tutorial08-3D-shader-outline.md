---
layout: post
title:  "中级Shader教程08 3D shader 总纲"
date:   2018-03-27 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial theory
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
//




### **1.概论**
### **shader中函数的基本理解**
 > .raymarching
 > .SDF
 > .noise3D
 > .fbm
 > .混合光栅和raymarching
 

### **3D技巧合集**  
> .raymarching 加速
> .SDF(sign distance field)
> .3D 空间划分:极坐标
> .3D 空间划分:笛卡尔坐标
> .FBM 用于高度
> .FBM 用于密度
> .多层透明混合
> .wave 合成的多种方式
> .法线柔和过渡
> .raymarching 中折射反射的实现
> .gadray
> .hack shader 效果的精髓:"凑"字决


### **范例中包含新的特性有** 

#### 1.Raymarching a simple scene
> .[raymarching][11]

#### 2.Raymarching 基本框架
> .[框架抽取][11]

#### 3.SDF提要
> .[SDF][13]
#### 4.BounceBalls
> .[笛卡尔坐标空间划分][15]
#### 5.Stars
> .[极坐标空间划分][16]
#### 6.Sky
>. [FBM 用于密度][17]
#### 7.Mountain
>. [FBM 用于高度][18]
#### 8.Lake
> .法线柔和过渡  
> .[raymarching 中折射反射的实现][19]
#### 9.Sea
> .[wave 合成的多种方式][20]
#### 10.Fog
>. [FBM 用于密度][22]
#### 11.Cloud
> .[FBM 用于密度多层透明混合][23]
#### 12.水底世界
> .godray
> ."凑"字决
#### 13.地平线
> ."凑"字决



----------

### **7.链接：**
- [本教程配套blog ][1]
- [本教程配套项目源码 ][2]
- [教程中抽取的RayMarching框架][3]

  [1]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [2]: https://github.com/JiepengTan/FishManShaderTutorial
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
  [13]: https://jiepengtan.github.io/2018/04/23/shader-tutorial10-SDF/
  [14]: https://jiepengtan.github.io/2018/04/23/shader-tutorial11-default-renderframe/
  [15]: https://jiepengtan.github.io/2018/04/23/shader-tutorial12-bounced-balls/
  [16]: https://jiepengtan.github.io/2018/04/23/shader-tutorial13-stars/
  [17]: https://jiepengtan.github.io/2018/04/23/shader-tutorial14-sky/
  [18]: https://jiepengtan.github.io/2018/04/23/shader-tutorial15-mountain/
  [19]: https://jiepengtan.github.io/2018/04/23/shader-tutorial16-lake/
  [20]: https://jiepengtan.github.io/2018/04/23/shader-tutorial17-sea/
  [21]: https://jiepengtan.github.io/2018/04/23/  [12]: shader-tutorial18-mutil_transparent_render/
  [22]: https://jiepengtan.github.io/2018/04/23/shader-tutorial19-fog/
  [23]: https://jiepengtan.github.io/2018/04/23/shader-tutorial20-cloud/
  [24]: https://jiepengtan.github.io/2018/04/25/shader-tutorial21-shader-tips-compute/

