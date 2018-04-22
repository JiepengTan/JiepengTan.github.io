---
layout: post
title:  "中级Shader教程13 Lake"
date:   2018-04-13 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial shader sky water lake mountain flare
mathjax: true
---
<p align="center"> <img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Stars/head.gif?raw=true" width="512"/></p>
<p align="center"></p>  



### 1.实现原理  
1.使用球坐标来进行空间划分  
2.对每个空间划分的grid产生hash  
3.根据hashID 定义星星的大小，闪烁周期，明暗程度等  
4.在grid绘制圆 用smoothstep 来控制圆的大小  


### 2.源码

1.单层FBM中不同的层之间移动速度随时间的偏移  

```c
// 通过rd 来进行空间划分 这样在根据相机进行改变
float3 Stars(in float3 rd,float den,float tileNum)
{
    float3 c = float3(0.,0.,0.);
    float3 p = rd;
    float SIZE = 0.5;
    //分多层
    for (float i=0.;i<3.;i++)
    {
        float3 q = frac(p*tileNum)-0.5;
        float3 id = floor(p*tileNum);
        float2 rn = hash33(id).xy;

        float size = (hash13(id)*0.2+0.8)*SIZE; 
        float demp = pow(1.-size/SIZE,.8)*0.45;
        float val = (sin(_Time.y*31.*size)*demp+1.-demp) * size;
        float c2 = 1.-smoothstep(0.,val,length(q));//画圆
        //随机显示 随着深度的层数的增加添加更多的星星 增加每个grid 出现星星的概率
        c2 *= step(rn.x,(.0005+i*i*0.001)*den);
        c += c2*(lerp(float3(1.0,0.49,0.1),float3(0.75,0.9,1.),rn.y)*0.25+0.75);//不同的亮度
        p *= 1.4;//增加grid密度
    }
    return c*c*.7;
}
```


### 2.unity shader源码
shader源码可以在这里[下载][1]


  [1]: https://github.com/JiepengTan/FishManShaderTutorial