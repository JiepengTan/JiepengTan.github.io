---
layout: post
title:  "中级Shader教程13 3d Sky"
date:   2018-04-16 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial shader sky noise cloud
mathjax: true
---
<p align="center"> <img src="http://127.0.0.1:4000/assets/img/blog/ShaderTutorial3D/Sky/head.gif" width="512"/></p>
<p align="center"></p>  


 


### 1.实现原理
1.使用FBM 来模拟基本的云层形状
2.在FBM的过程中对不同layer位置进行随时间不同程度的偏移


### 2.源码

1.单层FBM中不同的层之间移动速度随时间的偏移  
```c
float TimeFBM( float2 p,float t )
{
    float2 f = 0.0;
    float s = 0.5;
    float sum =0;
    for(int i=0;i<5;i++){
        p += t;//位置添加时间偏移
        t *=1.5;//每一层时间偏移不同 等到不同分层不同移速的效果
        f += s*tex2D(_NoiseTex, p/256).x; p = mul(float2x2(0.8,-0.6,0.6,0.8),p)*2.02;
        sum+= s;s*=0.6;
    }
    return f/sum;    
}
```

2.大的云层分布  

```c
float3 Cloud(float3 bgCol,float3 ro,float3 rd,float3 cloudCol,float spd, float layer)
{
    float3 col = bgCol;
    float time = _Time.y*0.05*spd;
    //对不同的大的分层 添加不同的基本高度
    for(int i=0; i<layer; i++){
        float2 sc = ro.xz + rd.xz*((i+3)*40000.0-ro.y)/rd.y;
        //颜色与背景色混合叠加
        col = lerp( col, cloudCol, 0.5*smoothstep(0.5,0.8,TimeFBM(0.00002*sc,time*(i+3))) );
    }
    return col;
}

```

3.其他部分的绘制  

```
// create by JiepengTan 2018-04-14  email: jiepengtan@gmail.com
float4 ProcessRayMarch(float2 uv,float3 ro,float3 rd,inout float sceneDep,float4 sceneCol)  {
    fixed4 sph = fixed4(0.0,0.0,0.0, 0.5);
    fixed3 col = fixed3(0.0,0.0,0.0);  
    float3 light1 = normalize( float3(-0.8,0.4,-0.3) );
    float sundot = clamp(dot(rd,light1),0.0,1.0);
   
    
    //基本的天空颜色过渡
    col = float3(0.2,0.5,0.85)*1.1 - rd.y*rd.y*0.5;
    col = lerp( col, 0.85*float3(0.7,0.75,0.85), pow( 1.0-max(rd.y,0.0), 4.0 ) );
    
    //太阳绘制 
    col += 0.25*float3(1.0,0.7,0.4)*pow( sundot,5.0 );
    col += 0.25*float3(1.0,0.8,0.6)*pow( sundot,64.0 );
    col += 0.2*float3(1.0,0.8,0.6)*pow( sundot,512.0 );
    // 云
    col = Cloud(col,ro,rd,float3(1.0,0.95,1.0),1,1);
    // 过滤掉地平线以下的部分
    col = lerp( col, 0.68*float3(0.4,0.65,1.0), pow( 1.0-max(rd.y,0.0), 16.0 ) );
    sceneCol.xyz = col;
    return sceneCol;
} 
```



### 2.unity shader源码
shader源码可以在这里[下载][1]


  [1]: https://github.com/JiepengTan/FishManShaderTutorial