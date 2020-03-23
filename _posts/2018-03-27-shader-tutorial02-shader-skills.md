---
layout: post
title:  "中级Shader教程02 shader技巧总览"
date:   2018-03-27 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial mathmatic shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
//





### 0.说明
本篇文章会随着教程的更新不断的更新和丰富

### 1.shader技巧总览
1. 空间划分
2. 基于格子的随机性
3. 分层
4. 类FBM框架 
  . 重复多次类似操作 并收集结果 val1,val2,val3 ...  
  . 将收集到的结果进行集合操作，如min,sum,avg等    
5. 法线柔和过渡
6. 多层透明混合
7. 基于范围的图像处理
8. 基于方向的图像处理

### 2.详细例子：
#### 1.空间划分  
1.[雪花][8]  
1.[Stars][16]  

#### 2.基于格子的随机性    
1.[雪花][8]  
2.[火焰粒子][9]  
3.[BounceBall][15]  

#### 3.分层     
1.[2D海洋][7]  
2.[雪花][8]  

#### 4.类FBM框架     
1.[Sky][17]  
2.[Mountain][18]  
3.[Lake][19]  
4.[Sea][20]  
5.[Fog][22]  
6.[Cloud][23]  

#### 5.法线柔和过渡    
1.[Mountain][18]  
2.[Lake][19]  
3.[Sea][20]  

#### 6.多层透明混合    
1.[Fog][22]  
2.[Cloud][23]  



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