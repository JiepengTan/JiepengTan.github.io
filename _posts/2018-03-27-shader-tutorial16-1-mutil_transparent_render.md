---
layout: post
title:  "中级Shader教程16 多层透明叠加渲染"
date:   2018-04-23 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial sea shader wave
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---

### **0.说在前面**
1.我们通常说起颜色通常说“这个颜色的rgb值是多少”，而不是说“这个颜色的rgba值是多少”,不说a是因为我们默认改颜色的a = 1.0，即是不透明的  
2.所以得提醒：使用rgba表示颜色时，我们最终看到的颜色不是rgb,而是rgb*a。这点对于理解多层透明混合很重要。  
3.这个代码将会在后面渲染云和狂风中出现，如果想要真正理解并灵活运用该渲染套路,请重视！  





### **1.从前往后混合：**
我们设前景色为fCol,背景色为bCol
一开始我们的可以拥有的目光为remainA = 1.0，看到到的颜色为sumCol = 0
迭代1：
    我们的目光透过前景色后：
    remainA1 = 1.0- fCol.a
    sumCol1 = fCol.rgb * fCol.a
    此时我们的目光只剩下remainA了
迭代2：
    当我们的目光透过前景色看背景色后，
    我们剩下的目光看到的颜色为
    viewCol = remainA1*(bCol.rgb*bCol.a)
    总的颜色为 sumCol2 = sumCol1 + viewCol
    这时候我们的目光还剩下 remainA2 = remainA1 *(1-bCol.a);// 
迭代3：
    设背景色的背景色为bbCol
    当我们的目光到达时，
    同样的我们计算：
    sumCol3 = sumCol + remainA2 *(bbCol.rgb*bbCol.a)
    remainA3 = remainA2 *(1-bbCol.a)

继续迭代下去：
将上述转换为代码为：

float3 sumCol = float3(0.,0.,0.);
float remainA = 1.;
for(int i=0;i<LayerNum;i++)
{
    float4 bCol = GetLayerBgCol(i);
    bCol.xyz = bCol.xyz * bCol.a;
    sumCol = sumCol + remainA*bCol.xyz;
    remainA = remainA * (1.-bCol.a);
}

将上述代码简化为：
float4 sumCol = float4(0.,0.,0.,0.);
for(int i=0;i<LayerNum;i++)
{
    float4 bCol = GetLayerBgCol(i);
    bCol.xyz = bCol.xyz * bCol.a;
    sumCol = sumCol + (1.-sumCol.a)*bCol;
}




### **2.从后往前混合：**
这个简单,因为最后面的背景色一定是不透明的(因为如果是透明的，则我们还可以透过它看到下一层).
所以从后叠加：设当前当前最后一层为bCol,前一层为fCol:
迭代1:
sumCol = sumCol + bCol.rgb*bCol.a + (1.0-bCol.a)*sumCol.rgb
        = bCol.rgb
remianA = 1.0-bCol.a 
        = 0;
迭代2:
sumCol = fCol.rgb*fCol.a + bCol.rgb*bCol.a*(1.0-fCol.a)
       = fCol.rgb*fCol.a + bCol.rgb*1.0*(1.0-fCol.a)
       = fCol.rgb*fCol.a + bCol.rgb*(1.0-fCol.a) // 经典颜色混合公式 src.rga*src.a + dst.rgb*(1-src.a)
此时
remainA = fCol.a - (fCol.a*bCol.a) ;
        = 0. 
说明叠加后颜色依旧为不透明的
重复之：
得到代码为：
float3 sumCol = float3(0.,0.,0.);
for(int i=0;i<LayerNum;i++)
{
    float4 bCol = GetLayerBgCol(-i);//负数表示倒数第多少层
    sumCol = bCol.rgb*bCol.a + sumCol.rgb*(1.0-bCol.a) 
}
到了这里得到了经典渲染模式.


- [本教程配套blog ][1]
- [本教程配套项目源码 ][2]
- [教程中抽取的RayMarching框架][3]

  [1]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [2]: https://github.com/JiepengTan/FishManShaderTutorial
  [3]: https://github.com/JiepengTan/Unity-Raymarching-Framework