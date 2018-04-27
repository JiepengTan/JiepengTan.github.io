---
layout: post
title:  "中级Shader教程16.3 3D云"
date:   2018-03-27 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial sea shader wave
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial3D/Cloud/head.gif?raw=true" width="512"></p>   





### 1.前置知识链接      
1.Noise和FBM请参考[这篇文章中给出的链接][4]       
2.[ranymarching 框架][5]     
3.[多层透明叠加渲染][6]  

### 1.分析&实现
#### 1.使用常规raymarching +倒退+ 多层叠加透明渲染  
1.常用FBM *分布限制来定义距离函数Map(pos)  
2.定义密度函数为 Den(pos) = clamp(-(Map(pos)) ,0.,1.)  
这样在raymarching hit的点p =(ro+t*rd)其实密度其实才刚开始，然后在 p沿射线倒退  
一定的距离，然后开始进行多层透明渲染。  

伪代码:  
```c 
float Map(float3 p){
    return  abs(p.y-cloudY)* cloudH // 大方向的限制
                +FBM(p*0.4) // 局部反密度分布
                    -0.4;//因为FBM ~ [0~1.] ,为了得到一个拥有“内部”这个概念的距离分布 所以应该减去一个值
}
float Den(float3 p){return clamp(-Map(p),0.,1.);
float Raycast(in float3 ro, in float3 rd)
{
    float precis = .1;//这里检测到快要接近 即可
    float h= 1.;
    float d = 0.;
    for( int i=0; i<_LoopNum.x; i++ )
    {
        if( abs(h)<precis || d>70. ) break;
        d += h;
        float3 pos = ro+rd*d;
        h = Map(pos) * .8;
    }
    return d;
}

float3 RenderCloud(float3 bgCol,float3 ro,float3 rd,float rz){
    float t = rz- 5.;
    float4 sum = float4( 0.0 , 0.0 , 0.0 , 0.0 );
    for( int i=0; i<128.; i++ )
    {
        if(rz.a > 0.99 || t > MAXT) break;

        float3 pos = ro + t*rd;
        float den = Den(pos);
        //直接作色 里面的云层颜色为黑色
        float4 col = float4(lerp( float3(.8,.75,.85), 
                                float3(.0,.0,.0), den ),den);
        //阴影 检测当前位置沿光照方向前面的点的密度来决定阴影程度
        float sh = smoothstep(0.2+moy*0.05,.0,mapV(pos+1.*lgt))*.85+0.15;
        col *= sh; 
        //常规 多层透明叠加渲染 
        col.a *= .9;
        col.rgb *= col.a;
        sum = sum + col*(1.0 - sum.a);
        t += max(.4,(2.-den*30.)*t*0.011);//根据密度步进
    }
    return  bgCol*(1.0-res.w) + res.xyz;
}
```
案例请参考：[nimitz 的 Cloud Ten][7]  

#### 2.密度函数+积分  

直接定义密度函数，然后多层叠加渲染    

```c
// 直接FBM 定义密度函数
float4 MapCloud(float3 pos){
    float d = 1.0-0.15*abs(10.8 - pos.y);//限制在y = 10.8 附近
    d -= 1.6 * FBM( pos*0.15 );//FBM定义密度函数
    d = clamp( d, 0.0, 1.0 );
    float4 res = float4(d,d,d,d);
    res.xyz = lerp( 0.8*float3(1.0,0.95,0.9), 0.2*float3(0.6,0.6,0.6), res.x );
    res.xyz *= 0.65;
    return res; 
}

float3 RenderCloud(float3 bgCol,float3 ro,float3 rd,float tmax){
    float4 sum = float4(0., 0., 0., 0.);
    float dif = clamp( dot(rd,_LightDir), 0.0, 1.0 );
    float t = 0.1;
    //直接积分渲染
    for(int i=0; i<64.; i++) {
        if( sum.w > 0.99 || t > tmax ) break;
        float3 pos = ro + t*rd;
        float4 col = MapCloud( pos );
        //col.xyz *= float3(0.4,0.52,0.6);
        //添加光照影响
        col.xyz += float3(1.0,0.7,0.4)*0.3*pow( dif, 6.0 )*(1.0-col.w);
        //在相机移动的时候可以平缓过渡
        col.xyz = lerp( col.xyz, bgCol, 1.0-exp(-0.0018*t*t) );

        //常规 多层透明叠加渲染 
        col.a *= 0.5;
        col.rgb *= col.a;
        sum = sum + col*(1.0 - sum.a);  
        t += max(0.1,0.3*t);
    }
    sum =  clamp( sum, 0.0, 1.0 );
    return bgCol*(1.0-sum.w) + sum.xyz;
}
```
案例请参考：[iq 的 Clouds][8]







- [本教程配套blog ][1]
- [本教程配套项目源码 ][2]
- [教程中抽取的RayMarching框架][3]

  [1]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [2]: https://github.com/JiepengTan/FishManShaderTutorial
  [3]: https://github.com/JiepengTan/Unity-Raymarching-Framework
  [4]: https://jiepengtan.github.io/2018/03/27/shader-tutorial01-base-math/
  [5]: https://jiepengtan.github.io/2018/04/22/shader-tutorial09-1-raymarch-framework/
  [6]: https://jiepengtan.github.io/2018/04/23/shader-tutorial16-1-mutil_transparent_render/
  [7]: https://www.shadertoy.com/view/XtS3DD
  [8]: https://www.shadertoy.com/view/XslGRr



