---
layout: post
title:  "中级Shader教程03 3D Raymarch框架"
date:   2018-04-10 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial 3D_Raymarch shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---

### **3D Raymarch框架** 
```c
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{   
    //1.uv空间重映射
    vec2 uv = RemapUV(fragCoord);
    
    //2.设置相机信息
    vec3 ro,rd;
    SetCammera(uv,ro,rd);
    
    //3.发射射线，计算与场景的碰撞点，并获取位置，材质信息
    vec2 ret = CastRay(ro,rd);
    float matId = ret.y;
    vec3 pos = ro+ret.x*rd;
    
    //4.计算碰撞点的法线信息    
    vec3 nor= normal( pos, rz );
    
    //5.使用步骤4获得的信息计算当前像素的的颜色值
    vec3 col = Shading( pos,nor,matId);
    
    //6.gamma 矫正
    col = pow( col, vec3(0.4545) );//unity 中不需要
}
```






###  1.uv空间重映射
这里目标是将uv空间设置为屏幕中心点为（0，0）,右上点坐标为（0.5*屏幕宽高比，0.5）的笛卡尔坐标系
```c
vec2 RemapUV(vec2 fragCoord){
    vec2 uv = fragCoord.xy / iResolution.xy;//此时uv空间为(0.0 ~ 1.0)
    uv = uv - 0.5;//此时uv空间为(-0.5 ~ 0.5)
    uv.x*=iResolution.x/iResolution.y;//将uv拉伸回正常比例
}
```

###  2.计算射线初始位置和方向
通常是模拟3D相机的近裁剪面来发射射线，当然发射的射线也可以根据需求自定义，比如：
1.nimitz 的 [Sirenian Dawn][1] 场景中为了得到类似星球的的效果，对v坐标根据u坐标进行响应的调整
> rd.y += abs(p.x*p.x*0.015);

2.将uv转为球坐标，可以模拟全视角摄像机的效果

```c
void SetCammera(in vec2 uv,out vec3 ro, out vec3 vec2rd){
    ro = GetCameraPos();//获取相机的位置 
    vec3 ta = GetTargetPos();//获取目标位置
    vec3 forward = normalize( ta - ro);//计算 forward 方向
    vec3 left = normalize(cross( vec3(0.0,1.0,0.0), forward ));//计算 left 方向
    vec3 up = normalize(cross(forward,left));////计算 up 方向
    const float zoom = 1.;
    rd = normalize( p.x*left + p.y*up + -zoom*forward );
}

```

### 3.光线追踪
这里通常使用迭代的方式执行光线追踪，但如果场景特殊，也可以直接使用公式计算出来进行加速
如：srtuss 的 [jetstream][2]  这个场景就使用了公式直接计算碰撞点

同样CastRay 中t的步进策略有多种

1.使用常数步进幅度

2.使用当前位置距离场景最近点的距离来作为步进幅度

3.根据当前光线离相机的位置来增加步进幅度，因为离相机太远可以适当的降低精度

可以根据自己场景的特点自己设计也行，在速度和精度之间权衡
```c
// 使用SDF或者使用公式直接定义场景中物体的形状
// 返回两个值 
//res.x = pos到当前场景所有物体中的最近的距离
//res.y = 当前场景中离pos最近的物体的材质ID
vec2 Map(vec pos){
    vec2 res = vec2( sdPlane( pos), 1.0 )；
    res = opU( res, vec2( sdBox( pos-vec3( 1.0,0.25, 0.0), vec3(0.25) ), 3.0 ) );
    ....
    return res;
}

#define MARCH_NUM 64 //最多光线检测次数
vec2 CastRay( in vec3 ro, in vec3 rd )
{
    float tmin = 1.0;
    float tmax = 20.0;
   
    float t = tmin;
    float m = -1.0;
    for( int i=0; i<MARCH_NUM; i++ )
    {
        float precis = 0.0005*t;
        vec3 pos = ro+rd*t;
        vec2 res = Map(pos);
        if( res.x<precis || t > tmax ) break;
        t += res.x;// 加速检测速度 这里可以有不同的策略
        m = res.y;//材质ID
    }
    if( t>tmax ) m=-1.0;
    return vec2( t, m );
}
```
### 4.计算法线
法线本就是场景中位置的导数
```c
vec3 calcNormal( in vec3 pos )
{
    vec3 eps = vec3( 0.0005, 0.0, 0.0 );
    vec3 nor = vec3(
        Map(pos+eps.xyy).x - Map(pos-eps.xyy).x,
        Map(pos+eps.yxy).x - Map(pos-eps.yxy).x,
        Map(pos+eps.yyx).x - Map(pos-eps.yyx).x );
    return normalize(nor);
}

```
### 5.计算物体颜色
在这里你可以计算 diffuse，specular,reflect,refract,occ,fresnel，shadow等效果   
当然这里只是利用了材质球ID,UV之类的没有使用，如果想要实现复杂的材质效果，需要在光线追踪计算的时候获取那些信息如UV，厚度等信息。具体你可以参考 hsiangyun 的[荷花][3] 

```c
vec3 Shading(vec3 pos, vec3 nor, float matId){
    // light：diffuse reflect
    // shadow
    // reflect
    // refract
    // occ
    // fresnel
    ....
    vec3 matCol = GetMatColor(ID);
    return matCol;
}
```
### 6.多批次处理
将主框架渲染结果到缓存中后，可以对缓存做一些计算，可以实现一些屏幕效果，如边缘检测，bloom，godrays 之类的效果,想怎么搞都行。 


具体范例，你可以参考shadertoy的任意 3D场景，或者你只熟悉Unity,可以在[这里下载][4]一些范例


  [1]: https://www.shadertoy.com/view/XsyGWV
  [2]: https://www.shadertoy.com/view/XlsGRs
  [3]: https://www.shadertoy.com/view/XsVGzm
  [4]: https://github.com/JiepengTan/FishManShaderTutorial