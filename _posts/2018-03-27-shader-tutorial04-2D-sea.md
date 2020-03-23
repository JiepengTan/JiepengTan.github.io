---
layout: post
title:  "中级Shader教程04 2D海洋"
date:   2018-03-27 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial sea wave shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
 <p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Sea/head.gif?raw=true" width="512"></p> 

**本篇主要技术点有：**  
1. sin波函数的理解和应用  
2. 极坐标+sin 对于波动圆环的实现  
3. 2D模拟3D中"分层"的概念  





### **1.原理：**

sin函数是周期函数，对于无限重复的现象是一种很好的模拟
类似傅里叶变换一样，复杂的函数可以用简单的sin cos 组合进行比逼近
波浪这种效果是一种典型。

### **2.分析：**
#### **1.云的实现**
1. 形状:多个不同大小，偏移的圆，通过mix的方式合成云朵  
2. 动态:简单的走UV  
当然你也可以使用类似太阳光晕的那种方式实现  

#### **2.太阳的实现**
1. 形状:光的边缘是通过极坐标中的角度带入多个不同频率的sin函数中是实现  
2. 动态:不同频率的sin函数不同的相位偏移速度来实现   

#### **3. 海洋的实现**
0. 原理：观察海洋，海浪的变化是连续的，随机的，而且是相似的。
这几个特点是不是非常像前面文章提到过的Noise的感觉，局部是连续的，但整体的感觉又是随机的。
很类似对吧，后面我们实现山脉(FBM)和3D海洋(FFT)的时候，再回过头来看待这两种方式，会发现他们的合成形式是一样的，这里我们使用多个频率，振幅不同的Sin函数海实现对波浪的模拟。
我们通过"多层"的方式来模拟3D效果,同样需要考虑透视对于图像大小，范围，清晰度的影响
1. 形态：分层实现，单层中叠加不同频率，振幅，偏移的方式sin函数
2. 动态：不同的层不同的频率，同一层中，不同频率之间拥有不同的相位偏移速度

### **3.代码实现：**

为了方便画圆，调整UV为(-0.5,-0.5,0.5,0.5)

#### **1.云朵**
通过不同的圆来合成云朵
```c
float Circle(fixed2 uv,fixed2 center,float size,float blur){
    uv = uv - center;
    uv /= size;
    float len = length(uv);
    return smoothstep(1.,1.-blur,len);
}
//简单直观的合成方式
float DrawCloud(fixed2 uv,fixed2 center,float size){
    uv = uv - center;
    uv /= size;
    float col = Circle(uv,fixed2(0.,0.),0.2,0.05);
    col =col *  smoothstep(-0.1,-0.1+0.01,uv.y);//将圆中不想要的的部分给剪切掉
    col += Circle(uv,fixed2(0.15,-0.05),0.1,0.05);
    col += Circle(uv,fixed2(0.,-0.1),0.11,0.05);
    col += Circle(uv,fixed2(-0.15,-0.1),0.1,0.05);
    col += Circle(uv,fixed2(-0.3,-0.08),0.1,0.05);
    col += Circle(uv,fixed2(-0.2,0.),0.15,0.05);
    return col;
}
float DrawClouds(fixed2 uv){
    uv.x += 0.03*_Time.y;
    uv.x = frac(uv.x+0.5) - 0.5;
    float col = DrawCloud( uv,fixed2(-0.4,0.3),0.2);
    col += DrawCloud( uv,fixed2(-0.2,0.42),0.2);
    col += DrawCloud( uv,fixed2(0.0,0.4),0.2);
    col += DrawCloud( uv,fixed2(0.15,0.3),0.2);
    col += DrawCloud( uv,fixed2(0.45,0.45),0.2);
    return col;
}
```

#### **2.太阳**
```c
float AngleCircle(fixed2 uv,fixed2 center,float size,float blur){
    uv = uv - center;
    uv /= size;
    float deg = atan2(uv.y,uv.x) + _Time.y * -0.1;//转化为极坐标
    float len = length(uv);//转化为极坐标
    //通过极坐标角度的方式来绘制波浪型的圆环
    float offs =( sin(deg*9)*3.+sin(deg*11+sin(_Time.y*6)*.5))*0.05;
    return smoothstep(1.+offs,1.-blur+offs,len);
}

fixed2 sunPos = fixed2(0.3,0.35);
fixed sun = Circle(uv,sunPos,0.06,0.05);//绘制太阳中心的圆
fixed sunCircle = AngleCircle(uv,sunPos,0.08,0.05);//绘制太阳光晕
col = lerp( col ,fixed3(0.9,0.6,0.15),sunCircle);//融合光晕和背景色
col = lerp( col ,fixed3(0.98,0.9,0.1),sun);//融合太阳和光晕

```

#### **3.海洋**
为了计算方便这里的UV范围为(0,0,1.0,1.0)

```c
fixed Wave(float layer,fixed2 uv,fixed val){
    float amplitude =  layer*layer*0.00004;//这些数值都是为了美术效果  怎么漂亮怎么来
    float frequency = val*200*uv.x/layer;
    float phase = 9.*layer+ _Time.z/val;
    return amplitude*sin(frequency+phase); 
}
fixed3 col = fixed3(0.0,0.0,0.0);
float num = 0.;
for (float i=1.; i < LAYER; i++) {
    //类似FBM的叠加方式，没加一层整幅下降一半，平率提升两倍左右，目的是
    //既可以用第一个函数控制大概的形状，又可以增加后面的函数添加高频的变化，方便控制细节
    //同样这些参数 是为了让函数的形状变得好看
    float wave = 2.*Wave(i,uv,1.)+Wave(i,uv,1.8)+.5*Wave(i,uv,3.);
    float layerVal = 0.7-0.03*i + wave;//控制波浪的高度
    if(uv.y >layerVal){
        break;
    }
    num = i;//计算所在层的ID
}
col = num*fixed3(0,.03,1);//计算每一层的基本颜色
col += (LAYER - num) * fixed3(.04,.04,.04);//颜色叠亮
```

#### **4.背景色处理**
```c
//在最高的一层海浪之上
if(num ==0){
    //添加海平面泛光
    float ry = Remap(0.7,1.0,1.0,0.0,uv.y);//0.7是最高海浪值的水平面
    col = lerp(fixed3(0.1,0.6,0.9),fixed3(0.1,0.7,0.9),ry);//简单的颜色渐变
    col += pow(ry,10.)*fixed3(0.9,0.2,0.1)*0.2;//让接近海平面的地方泛白 pow是为了控制影响范围
}
```

## [**配套视频**][40]  
- [本教程配套blog ][1]
- [本教程配套项目源码 ][2]
- [教程中抽取的RayMarching框架][3]

  [1]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [2]: https://github.com/JiepengTan/FishManShaderTutorial
  [40]:https://space.bilibili.com/308864667/channel/detail?cid=112754
  [3]: https://github.com/JiepengTan/Unity-Raymarching-Framework