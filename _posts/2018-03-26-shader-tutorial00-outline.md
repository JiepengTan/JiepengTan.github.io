---
layout: post
title:  "中级Shader教程00 总纲"
date:   2018-03-26 16:09:03
author: Jiepeng Tan
categories: 
 - shader tutorial
tags: shader_tutorial
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---




### **0.说在前面**

如果你觉得本教程有用，就去[github][1]中给我颗星星⭐吧.

### **1.内容**：
>1. 教程中会讲解在编写shader的常用技巧，以及在项目中如何使用这些shader
>2. 大量的实例如水，火，粒子，海洋，山脉，闪电等
>3. 一些shader实现的理论知识
>因为本人也会点特效制作，所以本教程会有比较多的描绘自然现象的shader，如熔岩，雪花，冰，水，火，粒子，海洋，山脉，闪电等.
>4. 已经抽取一个[RayMarching框架][13],更加方便编写raymarching shader

### **2.目录**

#### 1.**理论知识** 
 - [基本数学函数][4]
 - [shader技巧总纲][5]
 - [2D shader框架][6]
 - [3D raymarching框架][11]
 - [基本建模SDF][12]
 - 优化:用shader分摊CPU压力

----------


#### 2.**实例**
 1. **2D Shader基础**
    - [2D海洋][7]
    - [雪花][8]
    - [火焰粒子][9]
    - [熔岩][10]
    - 下雨
 2. **3D Shader**
    - [Unity 和 Raymarch 整合][11]
    - 天空
    - 地形
    - 大海
 3. **shader技术整合**
    - GameUI 血瓶
    - 荒原湖泊
    - 西湖

----------

### **6.部分效果图：**
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/BaseMath/head.gif?raw=true" width="256"></p> 
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Snow/head.gif?raw=true" width="256"></p> 
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/GameHPUI/head.gif?raw=true" width="256"></p>
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Sea/head.gif?raw=true" width="256"></p>
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/FireParticle/head.gif?raw=true" width="256"></p> 
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Rain/head.gif?raw=true" width="256"></p> 
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Sky/head.gif?raw=true" width="256"></p>
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Stars/head.gif?raw=true" width="256"></p> 
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Lake/head.gif?raw=true" width="256"></p>
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Fog/head.gif?raw=true" width="256"></p> 
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/JobSys/head.gif?raw=true" width="256"></p> 

<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Caustic/head.gif?raw=true" width="768"></p>
 
----------

### **7.链接：**
- [本教程配套项目源码 ][1]
- [本人shadertoy地址 ][2]
- [第一时间更新blog地址][3]
- 如果想学习哪种类型的shader，可以在[这里][14]留言,我优先出留言中的shader的教程

  [1]: https://github.com/JiepengTan/FishManShaderTutorial
  [2]: https://www.shadertoy.com/user/FishMan
  [3]: https://jiepengtan.github.io/
  [4]: https://jiepengtan.github.io/2018/03/27/shader-tutorial01-base-math/
  [5]: https://jiepengtan.github.io/2018/03/27/shader-tutorial02-shader-skills/
  [6]: https://jiepengtan.github.io/2018/03/27/shader-tutorial03-2D-shader-framework/
  [7]: https://jiepengtan.github.io/2018/03/27/shader-tutorial04-2D-sea/
  [8]: https://jiepengtan.github.io/2018/03/27/shader-tutorial05-2D-snow/
  [9]: https://jiepengtan.github.io/2018/03/27/shader-tutorial06-2D-fire-particle/
  [10]: https://jiepengtan.github.io/2018/03/27/shader-tutorial07-2D-lava/
  [11]: https://jiepengtan.github.io/2018/04/22/shader-tutorial09-1-raymarch-framework/
  [12]: https://jiepengtan.github.io/2018/04/23/shader-tutorial10-SDF/
  [13]: https://github.com/JiepengTan/Unity-Raymarching-Framework
  [14]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  