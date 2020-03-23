---
layout: post
title:  "中级Shader教程25 水专题02:两种涟漪实现方式"
date:   2018-04-26 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial theory shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Rain/head.gif?raw=true" width="512"></p>






### 1.实现
#### 1.空间划分+相邻grid之间采样实现
其中
```c
 y = sin(31.*t) * smoothstep(-0.6, -0.3, t) * smoothstep(0., -0.3,t)
```
图形解析为：  
<p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Ripple/math.gif?raw=true" width="256"></p>


```c
// create by JiepengTan 
// https://github.com/JiepengTan/FishManShaderTutorial2018-04-25 
// email: jiepengtan@gmail.com
// all right reserve
float _Ripple(float period,float spreadSpd,float waveGap,float2 uv,float rnd){
     // sample the texture
    const float WAVE_NUM = 2.;
    const float  CROSS_NUM = 1.0;
    float ww = -WAVE_NUM * .5 * waveGap;
    float hww = ww * 0.5;
    float freq = WAVE_NUM * PI2 / waveGap/(CROSS_NUM + 1.);
    float radius = (float(CROSS_NUM));
    float2 p0 = floor(uv);
    float sum = 0.;

    //多个格子中的波动全部累计起来
    for (float j = -CROSS_NUM; j <= CROSS_NUM; ++j){
        for (float i = -CROSS_NUM; i <= CROSS_NUM; ++i){
            float2 pi = p0 + float2(i, j);
            float2 h22 = Hash23(float3(pi,rnd));
            float h12 = Hash13(float3(pi,rnd));
            float pd = period*( h12 * 1.+ 1.);//让周期随机
            float time = ftime+pd*h12;//让时间偏移点  不会全部同时出现
            float t = fmod(time,pd);
            float spd = spreadSpd*((1.0-h12) * 0.2 + 0.8);//让传播速度随机
            float size = (h12)*0.4+0.6;
            float maxt = min(pd*0.6,radius *size /spd);
            float amp = clamp01(1.- t/maxt);
            float2 p = pi +  Hash21(h12 + floor(time/pd)) * 0.4;
            float d = (length(p - uv) - spd*t)/radius * 0.5;
            sum -= amp*sin(freq*d) *  smoothstep(ww*size, hww*size, d) *  smoothstep(0., hww*size, d);//让波动传播开来
        }
    }
    sum /= (CROSS_NUM*2+1)*(CROSS_NUM*2+1);
    return sum;
}

float Ripples(float2 uv ,float layerNum,float tileNum,float period,float spreadSpd,float waveGap){
    float sum = 0.;
    //分多层
    for(int i =0;i<layerNum;i++){
        sum += _Ripple(period,spreadSpd,waveGap,uv*(1.+i/layerNum ) * tileNum,float(i));
    }
    return sum ;
}
```

#### 2.图像处理+波动传播   
思想:波动传播     
优点:无论多么复杂的运动，时间复杂度都是O(1)  
缺点:需要两个缓存buffer来实现，同时效果和精度，受buffer的大小限制    

```c
// p11 是紫色坐标前两帧的值
// p10,p01,p21,p12对于的是自身坐标上下左右相邻像素的前一帧值
// The actual propagation:
   d += -p11 + (p10 + p01 + p21 + p12)*.5;
   d *= .999; // 衰减
```

可以参考[这里][4]


## [**配套视频**][40]  
- [本教程配套blog ][1]
- [本教程配套项目源码 ][2]
- [教程中抽取的RayMarching框架][3]


  [1]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [2]: https://github.com/JiepengTan/FishManShaderTutorial
  [40]:https://space.bilibili.com/308864667/channel/detail?cid=112754
  [3]: https://github.com/JiepengTan/Unity-Raymarching-Framework
  [4]: https://www.shadertoy.com/view/Xsd3DB
