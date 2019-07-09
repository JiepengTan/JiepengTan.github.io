---
layout: post
title:  "LockstepPlatform教程00 总纲"
date:   2019-07-06 16:09:03
author: Jiepeng Tan
categories: 
 - LockstepPlatform
tags: LockstepPlatform tutorial ECS lockstep
img_path: /assets/img/blog/LockstepPlatform
mathjax: true
---


# FishMan LockstepPlatform Tutorial

### **0.说在前面**
  朋友们,我又回来了,继上一次的 [shader教程][6] 后，这次推出的是游戏开发框架教程,目标受众是中高级程序员(有梦想的初级程序也行),当然了，这次会伴随着教学视频，毕竟视频+声音 学习效果会比文字好很多 :)

  本教程会同时伴随一个框架的诞生[LockstepPlatform][2]，该框架核心技术点是 帧同步 & ECS,目标是提供双端的帧同步解决方案
  本框架的初衷：
  1. 是给独立开发者使用,他们拥有多个小游戏，但是每个小游戏如果要联网，需要单独的服务器，如果采用非帧同步方案，如状态同步，会导致额外的网络逻辑代码，需要更长的开发周期，成本过高，综合多种原因导致很多小游戏都是单机。我想让独立游戏联机变成普遍现象。 

  **本教程内容涵盖：**
 
#### **1.库编写**
  - [数学库][14] 
  - [碰撞检测库2D][15] 
  - [碰撞检测库3D][15] 
  - [二进制序列化库][2]//在框架中没有独立出来 
  - [网络库的编写][2]//在框架中没有独立出来 
  - [(AI)NavMesh库的实现][18] 
  - [(AI)内存紧凑版行为树的实现][19] 
  - [ExcelParser 库的编写][2]//在框架中没有独立出来 
  - [ECS代码生成器][7] （兼容Entitas,UnityECS,LP Unsafe ECS,**当前版本仅兼容Entitas**） 
  - LP Unsafe ECS 库的编写//TODO 
  - AssetBundle资源管理 //TODO 
  - 物理库2D//TODO 
  - shader库的编写//TODO
  
#### **2.框架的演进**
  LockstepPlatform 从v0.1.0->v0.8.0 代码的框架的迭代过程

#### **3.优化技巧**
    1. 数据格式的设计
    2. 内存管理
    3. 资源管理
    4. 多线程
    5. shader优化

#### **4.普通游戏框架设计**
    本框架会同时涵盖多个demo,即使这些demo很简单,但还是会在这些demo中遵循相应的编程范式, 初,中级程序可以参考里面的编程技巧

### **1.项目目标**  

    1. 基于ECS快照回滚帧同步技术，提供一个游戏双端解决方案。
    2. 服务端：实现一个游戏平台，同时支持多个小游戏的运营，类似qq游戏平台
    3. 客户端：实现一个小引擎(除Audio,以及渲染器模块以及Editor，其他的都会基于确定性计算（整型）重写)

##### 目标用户  
   - 独立游戏开发者
   - 中小型公司
   - 喜欢学技术的朋友
   - 想使用部分模块的朋友

##### **当前版本(v0.3.0)**
    1. Excel 数据& 代码生成库 （done）
    2. 基于消息回调的网络消息库（done v0.6.0~v0.7.0 会增强为c# 基于await 异步回调 以及actor 模型）
    3. 客户端编程范式正式改为面向接口编程
    4. 服务器支持分布式(目前不成熟 仅用于测试双端游戏流程，v0.6.0~v0.7.0 会增强)

##### 版本（v0.2.0）
    1. ECS 目前采用Entitas (后续版本会改为基于c#指针&struct的版本。以及兼容Unity ECS 开发模式)
    2. (确定性)碰撞检测库 3D
    3. 序列化库(done)

##### 版本（v0.1.0）
    1. 基于整形的数学库 (done)
    2. (确定性)碰撞检测库 2D

##### 后续版本

###### v0.4.0(开发中)
    1. 基于c# 指针和struct实现的 内存紧凑版 行为树 (done)
    2. (确定性)NavMesh 库 (done) (TODO指针版本)
    3. ECL（a domain-specific language(DSL) for Entity component design）(解释器正在开发中)
    4. AssetBundleLoader (TODO)
    5. ECS代码生成器 (done) ECS 代码生成器 目前支持Entitas, 后续版本会同时支持生成Unity ECS代码，以及兼容Unsafe ECS代码  (新增)

###### v0.5.0（TODO）
    1. BehaviorTree 添加 Eidtor 支持
    2. ECL 
        - 兼容Unity ECS 
        - 开始支持基于c# 指针&struct 的版本
        - 支持导出为Excel 方便配制
    3. 2D 物理库
    4. NavMesh 指针化&struct 化
    5. AssetBundleLoader Editor


### **2.项目环境搭建**
  [环境搭建][5]

### **3.References**  
- ECS 回滚原型来自 UnityLockstep:[https://github.com/proepkes/UnityLockstep][11] 
- 当前网络库使用 LiteNetLib: [https://github.com/RevenantX/LiteNetLib][12] (v0.8.3 .NetCore)
- ECS Framework Entitas: [https://github.com/sschmid/Entitas-CSharp][13] (v1.13.0)
- 确定性 数学库: [https://github.com/JiepengTan/LockstepMath][14]  
- 确定性 碰撞检测库: [https://github.com/JiepengTan/LockstepCollision][15] 
- 确定性 NavMesh 库: [https://github.com/JiepengTan/LockstepPathFinding][18] 
- 内存紧凑版 行为树库: [https://github.com/JiepengTan/LockstepBehaviorTree][19] 
- ECL解释器: [https://github.com/JiepengTan/LockstepECL][20] //已经废弃 使用[ECS代码生成器][23]代替 因为c#完全可以胜任，但是可以用于学习 词法语法分析
- ECS代码生成器: [https://github.com/JiepengTan/LockstepECSGenerator][23]  
- 其他的库(Serialization，Logging,ExcelParser，Network)没有独立出项目来，但是库是作为单独的子模块位于本项目中[https://github.com/JiepengTan/LockstepPlatform][21]


### **4.2D Demo(FC Tank)**

#### ** Basic lockstep and rollback file**
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/Show/lockstepgifbig.gif?raw=true" width="512"></p>

#### ** Replay record file**
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/Show/lsp_recode_file2.gif?raw=true" width="940"></p>



----------

### **传送门**
- [本教程配套blog][1]
- [LockstepPlatform 框架源码 https://github.com/JiepengTan/LockstepPlatform][2]
- [2D demo 地址 https://github.com/JiepengTan/Lockstep_Demo2D_Tank][3]
- [3D demo 地址]//TODO
- [环境搭建][5]
- [FishMan shader教程 https://github.com/JiepengTan/FishManShaderTutorial][6]
- LockstepPlatform 帧同步 or ECS 技术交流qq群:**901118569**


 [1]: https://jiepengtan.github.io/2019/07/06/lp-tutorial00-outline/
 [2]: https://github.com/JiepengTan/LockstepPlatform
 [3]: https://github.com/JiepengTan/Lockstep_Demo2D_Tank
 [4]: https://github.com/JiepengTan/Lockstep_Demo2D_Tank
 [5]: https://jiepengtan.github.io/2019/07/06/lp-tutorial01-env-setup/
 [6]: https://github.com/JiepengTan/FishManShaderTutorial
 [7]: https://github.com/JiepengTan/LockstepECSGenerator
 [8]: https://jiepengtan.github.io/2018/03/26/shader-tutorial00-outline/
 [9]: https://github.com/JiepengTan/LockstepPlatform
 [10]: https://github.com/JiepengTan/LockstepPlatform
 [11]: https://github.com/proepkes/UnityLockstep
 [12]: https://github.com/RevenantX/LiteNetLib
 [13]: https://github.com/sschmid/Entitas-CSharp
 [14]: https://github.com/JiepengTan/LockstepMath
 [15]: https://github.com/JiepengTan/LockstepCollision
 [16]: https://github.com/JiepengTan/LockstepPlatform/releases
 [17]: https://github.com/sschmid/Entitas-CSharp/releases
 [18]: https://github.com/JiepengTan/LockstepPathFinding
 [19]: https://github.com/JiepengTan/LockstepBehaviorTree
 [20]: https://github.com/JiepengTan/LockstepECL
 [21]: https://github.com/JiepengTan/LockstepPlatform
 [22]: https://www.bilibili.com/video/av55450233
 [23]: https://github.com/JiepengTan/LockstepECSGenerator
 
 