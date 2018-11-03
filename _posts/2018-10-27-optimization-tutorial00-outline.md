---
layout: post
title:  "游戏优化教程00 总纲"
date:   2018-11-03 16:09:03
author: Jiepeng Tan
categories: 
 - GameOptimization
tags: game_optimization_tutorial
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---

# FishMan Game Optimization Tutorial

### **1.内容简介**：
>1. 游戏优化中的通用技巧和思想 
>2. 涵盖通用的资源管理设计模式,多线程,序列化框架等 
>3. 游戏中不同模块的优化分析，以及优化技巧 
>4. 游戏框架的设计 
>5. 通用库的抽取 


### **2.目录**
#### 1.**优化模块** 

##### 1. CPU:
 - [多线程][5]
 - [配置读取]
 - [Cache模式]
 - [AI:寻路]
 - [AI:行为树]
##### 2. 内存:
 - [可伸缩对象池]
 - [基于引用计数的自动化AB管理]
 - [基于分区的地图资源管理]
 - [压缩]
 - [美术资源制作规范]
##### 3. GPU：
 - [LOD的正确使用]
 - [阴影的优化]
 - [动态合批处理]
##### 4. IO：
 - [异步加载的使用]
 - [网络IO的优化]
 
#### 2.**框架设计** 
##### 1 .模块的设计：
 - [模块的设计：资源管理]
 - [模块的设计：辅助系统]
 - [模块的设计：战斗系统]
##### 2. 编程技巧：
 - [编程技巧：逻辑与渲染分离]
 - [编程技巧：Broken层抽象服务]
 - [编程技巧：反射]
 - [编程技巧：代码生成]

### **3.源代码：**
- [多线程][6]

### **4.其他：**

**1.一个人的能力和见识毕竟有限，对于框架的设计如果有好的建议或者参考项目源码，可以给我留言，或者给我发邮件JiepengTan@gmail.com，非常感谢。**
**2.如果发现了我的源码中的bug，你可以在对应的文章下面留言，或者给我邮件，我会非常感谢并尽快的修复**

- [本教程配套blog ][1]
- [我的另外一个教程：shader中级教程][2]
- [个人的github][3]
- [第一时间更新blog地址][4]

  [1]: https://github.com/JiepengTan/FishManShaderTutorial
  [2]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [3]: https://github.com/JiepengTan
  [4]: https://jiepengtan.github.io/ 
  [5]: https://jiepengtan.github.io/2018/11/03/optimization-tutorial01-mutil-thread/
  [6]: https://github.com/JiepengTan/ThreadPattern_Master-Slaver
  