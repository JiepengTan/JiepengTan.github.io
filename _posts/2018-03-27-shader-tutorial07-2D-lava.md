---
layout: post
title:  "中级Shader教程07 熔岩Lava"
date:   2018-03-27 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial snow grid shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
 <p align="center">
 <img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Lava/head.gif?raw=true" width="512"></p>

**本篇主要技术点有：**  
1. Noise的应用  
2. FBM的基本应用
3. 颜色空间映射





### 1.实现流程
1.单层noise分布  
 <p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Lava/noise.jpg?raw=true" width="256"></p>
2.多层叠加，每层noise添加位移和旋转  
 <p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Lava/fbm.jpg?raw=true" width="256"></p>
3.色温映射  
 <p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Lava/head.gif?raw=true" width="256"></p>
### 2.实现
#### 1.基本形态：  
1.通过FBM构造基本形态，在FBM添加点变化  
>1.每一层的移动速度不一样    
>2.每层的旋转不一样    

2.FBM的其他的应用还有  
>1.动态雾的密度    
>2.云层的密度  
>3.地形高程图  
>4.大理石纹理    
>5.3D动态云(fake)的实现原理和这里类似.不过没有添加旋转   

```c
float _Noise( in float2 x ){ return VNoise(x*0.75);}

float2 Gradn(float2 p)
{
	float ep = .09;
	float gradx = _Noise(float2(p.x+ep,p.y))-_Noise(float2(p.x-ep,p.y));
	float grady = _Noise(float2(p.x,p.y+ep))-_Noise(float2(p.x,p.y-ep));
	return float2(gradx,grady);
}
float FlowFBM(in float2 p)
{
	float z=2.;
	float rz = 0.;
	float2 bp = p;
	for (float i= 1.;i < 9.;i++ )
	{
		//让不同的层都添加不同的运动速度 形成一种明显的 层次感
		p += ftime*.0006;
		bp += ftime*0.00006;
		//获取梯度
		float2 gr = Gradn(i*p*1.54+ftime*.14)*4.;
		//添加旋转 让不同的图层拥有不同的旋转速度，形成整体有扭曲的感觉
		float2x2 rot = Rot2DRad(ftime*0.6-(0.05*p.x+0.07*p.y)*30.);
		gr = mul(rot,gr);
		p += gr*.5;
		//FBM实现
		rz+= (sin(_Noise(p)*7.)*0.5+0.5)/z;
		//插值调整每层之间效果
		p = lerp(bp,p,.77);
		//FBM 常规操作
		z *= 1.4;
		p *= 2.;
		bp *= 1.9;
	}
	return rz;	
}
```

#### 2.颜色映射  

```
// ...... Taken from https://www.shadertoy.com/view/MdBSRW
float3 Blackbody(float t)
{
	const  TEMPERATURE  = 2200.0;
	t *= TEMPERATURE;

	float u = ( 0.860117757 + 1.54118254e-4 * t + 1.28641212e-7 * t*t ) 
			/ ( 1.0 + 8.42420235e-4 * t + 7.08145163e-7 * t*t );

	float v = ( 0.317398726 + 4.22806245e-5 * t + 4.20481691e-8 * t*t ) 
			/ ( 1.0 - 2.89741816e-5 * t + 1.61456053e-7 * t*t );

	float x = 3.0*u / (2.0*u - 8.0*v + 4.0);
	float y = 2.0*v / (2.0*u - 8.0*v + 4.0);
	float z = 1.0 - x - y;

	float Y = 1.0;
	float X = Y / y * x;
	float Z = Y / y * z; 

	float3x3 XYZtoRGB = float3x3(3.2404542, -1.5371385, -0.4985314,
						-0.9692660,  1.8760108,  0.0415560,
							0.0556434, -0.2040259,  1.0572252);

	return max(float3(0.0,0.,0.), mul(XYZtoRGB,float3(X,Y,Z)) * pow(t * 0.0004, 4.0));
}
```

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/b/ba/PlanckianLocus.png/300px-PlanckianLocus.png" width="256">  

空间映射流程:  
温度 -> uv -> xy -> XYZ -> RGB  
具体原理参考：[色温映射][4]  
更多熔岩实现可以参考：  
[这里][5]  
 <p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Lava/ref1..jpg?raw=true" width="256"></p>  

和[这里][6]  
 <p align="center"><img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/ShaderTutorial2D/Lava/ref2.jpg?raw=true" width="256"></p>  

#### 3.整合在一起
```c
float3 ProcessFrag(float2 uv)
{
	uv*= _GridSize;
	float val = FlowFBM(uv);
	val = Remap(0.,1.,0.6,1.,val);
	float3 col = Blackbody(val);
	return col;
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
  [4]: https://en.wikipedia.org/wiki/Color_temperature
  [5]: https://www.shadertoy.com/view/MdBSRW
  [6]: https://www.shadertoy.com/view/XttSRs