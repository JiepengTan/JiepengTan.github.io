---
layout: post
title:  "游戏优化教程01 多线程"
date:   2018-11-03 16:09:03
author: Jiepeng Tan
categories: 
 - GameOptimization
tags: game_optimization_tutorial_mutilThread
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---

@[TOC]( FishMan优化教程总纲)

### **1.问题&挑战**：
在相对大型的游戏中，总是会碰到需要CPU占用时间长的任务,为了避免卡顿,可能需要进行分帧处理，在Unity中通常使用协程，但是需要协程本质上也是运行在主线程中，它依然会占用大量的主线程时间分片.所以使用多线程，但是多线程需要考虑下面几个因素：

>1.游戏的很多逻辑都是基于单线程开发的，所以使用多线程的时候，任务完成时，回调必须在主线程中回调，不然容易引起线程安全的问题。 
>2.游戏的有时候常常必须运行在主线程中，如需要使用实例化GameObjec,需要创建新的游戏资源如Texture,Mesh等，这些在Unity中必须运行在主线程中. 
>3.任务有时候需要中断，如一些先决条件的改变 
>4.如果我们的游戏是帧同步游戏，为了保证逻辑的一致，有些任务所有客户端都必须在同一个逻辑帧中完成。而这就要求其不能运行在其他线程。 
>5.当然主线程不能被挂起，否则游戏就卡死了. 
>6.游戏中不能到处充满Lock,Unlock的代码，不然维护成本过高 

所以线程框架必须具备以下几个特点：
>1.任务完成时候的回调必须在主线程中。 
>2.任务的某些阶段必须可以在主线程中运行 
>3.任务可以被取消 
>4.有些任务必须只能在当前帧中完成 
>5.主线程不能被其他线程给挂起 
>6.性能要好，因为引入多线程的原因就是为了性能 

### **2.设计**
基于上述几个需求:
>1.在没有任务的情况下，那些线程不应该占用CPU时间分片，所以应该被挂起。
需要使用信号量实现 
>2.我们拥有两种不同的运行环境，主线程和非主线程，但是为了同时满足上面的需求1,2需要一个运行在主线程的MainQueue,多线程本身就要求拥有一个运行在其他线程的AnyThreadQueue,需求4,所以我们还需要一个CurFrameQueue,这些队列都必须是线程安全的，所以必须加锁
而BlockQueue可以很好的满足上面的加锁和信号量的需求 
>3.//TODO 

### **3.源代码：**
- [多线程框架][6]

### **4.其他：**
- [本教程配套blog ][1]
- [我的另外一个教程：shader中级教程][2]
- [个人的github][3]
- [第一时间更新blog地址][4]

  [1]: https://github.com/JiepengTan/FishManShaderTutorial
  [2]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [3]: https://github.com/JiepengTan
  [4]: https://jiepengtan.github.io/ 
  [5]: https://github.com/JiepengTan/ThreadPattern_Master-Slaver
  [6]: https://github.com/JiepengTan/ThreadPattern_Master-Slaver
  