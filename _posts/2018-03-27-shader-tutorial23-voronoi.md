---
layout: post
title:  "中级Shader教程23 voronoi算法"
date:   2018-04-26 16:09:03
author: Jiepeng Tan
categories: 
- shader tutorial
tags: shader_tutorial theory shader
img_path: /assets/img/blog/ShaderTutorial2D/Snow
mathjax: true
---
<p align="center"> <img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/Base/Voronoi/head.gif?raw=true" width="256"></p>




### 1.基本Voronoi算法
1.在空间中摆放众多的样本点 points
2.对空间中的每个像素pixel，计算pixel到points的最短距离 minDist
3.根据minDist进行着色

伪代码：
```
float3[] points = AllocMemory();
UpdatePointPosition(points);
foreach pixel in pixels do
    float minDist = MAX_DIST;//100000
    float minPoint = 0;
    foreach point in points do 
        flaot dist = Distance(pixel,point);
        if(dist < minDist)
            minDist = dist;
            minPoint = point;
return CalcColor(minDist,minPoint);
```
时间复杂度为O(n^2)


### 2.Voronoi算法优化：
将空间grid化，让后对每一个grid,分配point 数据  
在2D版本中，其实只需要对9个格子计算，就可以得到最短距离  
时间复杂度为O(n)  

伪代码：  
```
float3[] points = AllocMemory();
UpdatePointPosition(points);
foreach pixel in pixels do 
    float minDist = MAX_DIST;//100000
    float minPoint = 0;
    Point[9] beiborPoints = GetNeiborGridPoints(); 
    foreach point in beiborPoints do 
        flaot dist = Distance(pixel,point);
        if(dist < minDist)
            minDist = dist;
            minPoint = point;
return CalcColor(minDist,minPoint);
```
### 3.算法拓展：
1.将最短距离改为第二短距离，第三短距离 ... 不同的条件可以得到不同的效果  
2.如上图voronoi得到的效果是晶格化，那么我们可以计算每个点到这些晶格化的“边”的最短距离，然后利用 这个结果进行着色会有不同的效果。  
3.将这个2D算法扩散到3D版本，不过周边的格子由9变为27  

### 4.更多信息可以参考
我的这篇关于Caustic效果的实现  
[The Book of Shaders][4]
iq大神的[blog1][5] [blog2][6]

### 5.unity shader源码
```c
// create by JiepengTan 2018-04-12  email: jiepengtan@gmail.com
Shader "FishManShaderTutorial/Voronoi"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
        _TileNum ("TileNum", float) = 5
    }

    SubShader
    {
        Tags { "RenderType"="Opaque" }
        LOD 100

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"
            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 vertex : SV_POSITION;
            };
            
            sampler2D _MainTex;
            float4 _MainTex_ST;
            float _TileNum ; 

            #define HASHSCALE3 float3(.1031, .1030, .0973)

            v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                return o;
            }
            float2 hash22(float2 p)
            {
                float3 p3 = frac(float3(p.xyx) * HASHSCALE3);
                p3 += dot(p3, p3.yzx+19.19);
                return frac((p3.xx+p3.yz)*p3.zy);

            }
            float wnoise(float2 p,float time) {
                float2 n = floor(p);
                float2 f = frac(p);
                float md = 5.0;
                float2 m = float2(0.,0.);
                for (int i = -1;i<=1;i++) {
                    for (int j = -1;j<=1;j++) {
                        float2 g = float2(i, j);
                        float2 o = hash22(n+g);
                        o = 0.5+0.5*sin(time+6.28*o);
                        float2 r = g + o - f;
                        float d = dot(r, r);
                        if (d<md) {
                            md = d;
                            m = n+g+o;
                        } 
                    }
                }
                return md;
            }
            fixed4 frag (v2f i) : SV_Target
            {
                float2 uv = _TileNum * i.uv;
                float time = _Time.y;
                float val = wnoise(uv,time);
                
                return float4(val,val,val,1.0);
            }           
            ENDCG
        }
    }
}

```

- [本教程配套blog ][1]
- [本教程配套项目源码 ][2]
- [教程中抽取的RayMarching框架][3]


  [1]: https://blog.csdn.net/tjw02241035621611/article/details/80038608
  [2]: https://github.com/JiepengTan/FishManShaderTutorial
  [3]: https://github.com/JiepengTan/Unity-Raymarching-Framework
  [4]: http://thebookofshaders.com/12/
  [5]: http://iquilezles.org/www/articles/voronoilines/voronoilines.htm
  [6]: http://www.iquilezles.org/www/articles/smoothvoronoi/smoothvoronoi.htm