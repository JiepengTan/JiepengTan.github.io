---
layout: post
title:  "中级Shader教程10 shader建模工具--SDF "
date:   2018-04-23 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial theory shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/SDF/head.gif?raw=true" width="512"></p> 






### 1.作用  
SDF (Sign Distance Functions)主要思想是计算点到目标模型的最近距离.  
在RayMarching中，如果已知射线点到场景中的左右物体的最短距离，就可以知道我们是否已经碰到的了物体，如果没有碰到场景，可以利用这个信息优化下一步步进的距离。  


### 2.概要  
代码包含在**SDF.cginc**中
基本函数实现来自Inigo's Quilez大神的[blog][7]

整个SDF建模API，包含4部分
>1.基本模型(如球长方体，圆锥等) 
2.集合操作(并集,交集,差集)  
3.Transform 操作(位移，旋转，缩放)  
4.变形操作(Twist,blend)  
5.**镜像复制** (repeat SDF特性)  
6.曲线相关(Bezier)  

熟悉3D建模工具如3ds max之类的工具，将会发现，这些函数基本上建模工具也有，不过这些建模工具额外还包含了顶点，边，面的操作。  

顺便提一下，在raymarching中对**不规律**事物的建模是低效的。在游戏中不建议使用。

### 3.API  
代码位于**SDF.cginc**文件中  
API 沿袭iq的版本.  
>1.基本模型  
SdXxx形式  
2.集合操作  
OpU,OpI,OpS  
3.Transform  
OpMove, OpRot,OpScale  
4.变形操作  
OpTwist,OpBend  
5.镜像操作  
OpRep OpRepX  
6.曲线  
SdBezier  


### 4.SdBezier曲线  
其中的一种实现方式的思想：[来自这里][4]  
bezier公式 t~[0,1]  
<p align="center">P(t) = (1-t)²P0 + 2t(1-t)P1 +t²P2</p>   
求导  
<p align="center">dP/dt(t) = -2(1-t)P0 + 2(1-2t)P1 + 2tP2</p>   
简化为  
<p align="center">dP/dt(t) = 2(A+Bt)</p>   
其中 A = (P1-P0) B = (P2-P1-A)  

<p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/SDF/bezier.jpg?raw=true" width="512"></p>     
如图，点M到曲线的最近点P,设点P处的切线为T = dp/dt(t)，则向量MP一定垂直于T即   
<p align="center">dot(MP,T)= 0</p>   
展开  
<p align="center">(M - (1-t)²P0 + 2t(1-t)P1 +t²P2).(A+Bt) = 0</p>  
继续展开  
<p align="center">at3 + bt² + ct + d = 0</p>  
其中  
<p align="center">M' = P0-M,a = B², b = 3A.B, c = 2A²+M'.B, d = M'.A</p>  
接下来是一元三次方程的求解，有公式可以直接求得:  
具体可以参考 Mathematics for 3D Game Programming and Computer Graphics (Third Edition)中的 6.1.2 Cubic Polynomials  
或者自己去google,最后获得的三个解中找到一个在范围[0,1]中的解即可  

具体实现cg版本代码请看**SDF.cginc**  
glsl版本：  
2D版本的请看[这里][5]  
3D版本的请看[这里][6]  

## [**配套视频**][40]  
- [本教程配套blog ][1]
- [本教程配套项目源码 ][2]
- [教程中抽取的RayMarching框架][3]

  [1]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [2]: https://github.com/JiepengTan/FishManShaderTutorial
  [40]:https://space.bilibili.com/308864667/channel/detail?cid=112754
  [3]: https://github.com/JiepengTan/Unity-Raymarching-Framework
  [4]: http://blog.gludion.com/2009/08/distance-to-quadratic-bezier-curve.html
  [5]: https://www.shadertoy.com/view/ltXSDB
  [6]: https://www.shadertoy.com/view/ldj3Wh
  [7]: http://iquilezles.org/www/articles/distfunctions/distfunctions.htm


