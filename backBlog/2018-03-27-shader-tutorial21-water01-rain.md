---
layout: post
title:  "中级Shader教程24 Ripple 两种雨实现方式"
date:   2018-04-26 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial theory shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---

<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Rain/head.gif?raw=true" width="768">






### 1.思想
1.和下雨特效制作一样，通过UV拉伸来呈现速度感，来表现雨

### 2.实现

#### 1.简单版本  
```c
float2 ruv = uv / float2(_ScreenParams.x/_ScreenParams.y,1.0);//原始uv
//宽高比正常uv
uv = (ruv * 2.0 - 1.0) *  float2(_ScreenParams.x/_ScreenParams.y,1.0);
float2 st =  uv * float2(.5+(ruv.y+1.0)*0.5, .03)+float2(ftime*.2-ruv.y*.1, ftime*.2);
//拉伸实现
float f = tex2D(_NoiseTex, st).y * tex2D(_NoiseTex, st*.773).x * 1.55;
f = clamp(pow(abs(f), 23.0) * 13.0, 0.0, (ruv.y-.2)*.14);
col += float3(f,f,f); 
```

#### 2.有光照的版本  
```c
float3 hitPos = RayCastScene();
float dis = 1.;
 for (int i = 0; i < LayerNum; i++)
{
    float3 plane = vCameraPos + originalRayDir * dis;
    //判定是否被雨阻挡
    if (plane.z < hitPos.z)
    {
        //根据z值缩小雨滴  透视模拟
        float f = pow(dis, .45)+.25;
        float2 st =  f * (uv * float2(1.5, .05)+float2(-ftime*.1+uv.y*.5, ftime*.12));
        // 拉伸uv后采集NoiseTex
        f = (tex2D(_NoiseTex, st * .5, -99.0).x + tex2D(_NoiseTex, st*.284, -99.0).y);
        // 让雨在离地面的地方变得透明
        f = clamp(pow(abs(f)*.5, 29.0) * 140.0, 0.00, uv.y*.4+.05);
        float3 bri = float3(.25);
        //光照处理
        for (int t = 0; t < NUM_LIGHTS; t++){
            float3 v3 = lightArray[t].xyz - plane.xyz;
            float l = dot(v3, v3);
            l = max(3.0-(l*l * .02), 0.0);
            bri += l * lightColours[t];
        }
        col += bri*f;
    }
    dis += 3.5;
}
col = clamp(col, 0.0, 1.0);

```

#### 3.raymarching版本
　　先将空间进行grid划分(类似[BouncedBalls][7])，让后使用类似Voronio实现方式对空间中的绘制长方体，加上前面提到的[多层透明混合][6]也可实现雨效果，不过性能会较为损伤


- [本教程配套blog ][1]
- [本教程配套项目源码 ][2]
- [教程中抽取的RayMarching框架][3]


  [1]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [2]: https://github.com/JiepengTan/FishManShaderTutorial
  [3]: https://github.com/JiepengTan/Unity-Raymarching-Framework
  [4]: https://jiepengtan.github.io/2018/03/27/shader-tutorial01-base-math/
  [5]: https://jiepengtan.github.io/2018/04/22/shader-tutorial09-1-raymarch-framework/
  [6]: https://jiepengtan.github.io/2018/04/23/shader-tutorial18-mutil_transparent_render/
  [7]: https://jiepengtan.github.io/2018/04/23/shader-tutorial12-bounced-balls/
