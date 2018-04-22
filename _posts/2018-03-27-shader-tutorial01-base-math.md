---
layout: post
title:  "中级Shader教程01 基础函数"
date:   2018-03-27 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial mathmatic shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
<p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/BaseMath/head.gif?raw=true" width="512"></p> 




以下实例的图片依次对上面图片中应笛卡尔坐标行列。
### **shader中函数的基本理解**  
 > 1.smoothstep 获取较为平滑的过渡效果length(uv) 降维效果   
 > 2.length(uv)将2D的转换为1D    
 > 3.atan(u,v) 获取角度 配合length 可以得到极坐标   
 > 4.pow(f,n) 将曲线变化变得平滑或尖锐  
 > 5.sin cos 周期函数，用于实现周期的效果(如来回移动，循环的移动等)  
 > 6.hash 获取标志ID，常用于基于空间划分的效果的实现  
 > 7.noise 同时具有随机性和连续性的函数   
 > 8.fbm 在基本函数的基础上，不断的叠加高频信息，丰富细节  


假设uv 是 (-0.5,-0.5) 到(.5,0.5)

**1.smoothstep eg:绘制smoothstep曲线**  
f(x) = x*x*(3.0-2.0*x)
具有淡入淡出效果
```c
float3 DrawSmoothstep(float2 uv){
    uv+=0.5;
    float val = smoothstep(0.0,1.0,uv.x);
    val = step(abs(val-uv.y),0.01); 
    return float3(val,val,val);
}
```

**2.length eg:绘制圆点**  
```c
float3 DrawCircle(float2 uv){
    float val = (1.0-length(uv)*2);
    val = smoothstep(0.0,0.1,val);
    return float3(val,val,val);
}
```

**3.atan eg:绘制花环**  
```c
float3 DrawFlower(float2 uv){
    float deg = atan2(uv.y,uv.x) + _Time.y * -0.1;
    float len = length(uv)*3.0;
    float offs = abs(sin(deg*3.))*0.35;
    return smoothstep(1.+offs,1.+offs-0.05,len);
}
```

**4.5.pow eg:让圆变得更虚or实**  
```c
float3 DrawWeakCircle(float2 uv){
    float val = clamp((1.0-length(uv)*2),0.,1.);
    val = pow(val,2.0);
    return float3(val,val,val);
}
float3 DrawStrongCircle(float2 uv){
    float val = clamp((1.0-length(uv)*2),0.,1.);
    val = pow(val,0.5);
    return float3(val,val,val);
}
```

**6.sin eg:跳动的小球**  
sin函数给2D小球y轴上的移动  
```c
float3 DrawBounceBall(float2 uv){
    uv*=4.;//uv 空间[-2~2]
    uv.y+=sin(ftime*PI);
    float val = clamp((1.0-length(uv)),0.,1.);
    val = smoothstep(0.,0.05,val);
    return float3(val,val,val);
}
```

**7.hash eg:不同颜色的格子**  
本教程中使用的Hash库来自Dave_Hoskin 
具体实现见Hash.cginc  
这里给出另外一种使用三角函数实现的Hash：  
```c
fixed2 Rand22(fixed2 co){
    fixed x = frac(sin(dot(co.xy ,fixed2(122.9898,783.233))) * 43758.5453);
    fixed y = frac(sin(dot(co.xy ,fixed2(457.6537,537.2793))) * 37573.5913);
    return fixed2(x,y);
}
```
fract(sin(x*bigVal1)*bigVal2)产生的数比较随机的原因是：   
sin(x*bigVal1) 会将x值的变化波动被放大，且随机，设这个值的变化为detX,则 frac(detX*bigVal2) 会让本来的小的波动再次放大，然后fract会让整体的数的变化受影响的因素变多，所以得到的了一个较为随机的值.  
更多的说明请参考[the book of shader][9]  
```c
float3 DrawRandomColor(float2 uv){
    uv+=0.5;
    uv*=4.;
    return Hash32(floor(uv));
}
```

**8.noise eg:Perlin Noise展示**  
本教程中使用的Noise库来自ShaderToy多位作者,具体实现见Noise.cginc  

本教程不再覆盖noise等基础函数的实,原理请参考[candycat这篇blog][4]  
更多实例可以参考[the book of shader][5]  
带有法线的noise函数的推导请看[iq][6]和[Milo][7]  

```c
float3 DrawRandomColor(float2 uv){
    uv+=0.5;
    uv*=4.;
    return Hash32(floor(uv));
}
```

**9.fbm eg:PerlinNoise FBM展示**  
FBM更多实例请参考[the book of shader][8]  

```c
float3 DrawRandomColor(float2 uv){
    uv+=0.5;
    uv*=4.;
    return Hash32(floor(uv));
}
```




- [本教程配套项目源码 ][1]
- [本人shadertoy地址 ][2]
- [第一时间更新blog地址][3]

  [1]: https://github.com/JiepengTan/FishManShaderTutorial
  [2]: https://www.shadertoy.com/user/FishMan
  [3]: https://jiepengtan.github.io/
  [4]: https://blog.csdn.net/candycat1992/article/details/50346469
  [5]: http://thebookofshaders.com/11/
  [6]: http://iquilezles.org/www/articles/gradientnoise/gradientnoise.htm
  [7]: https://stackoverflow.com/questions/4297024/3d-perlin-Noise-analytical-derivative
  [8]: http://thebookofshaders.com/13/
  [9]: http://thebookofshaders.com/10/
