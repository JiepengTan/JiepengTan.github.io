
**资源管理**
**1.原理**
1.在Unity 中，游戏Build之后的资源概念有三种:
>.GameObject (Go)
.Prefab (Fab)
.AssetBundle(AB)

资源对于的创建和销毁的API为：
>.GameObject: GameObject.Instancate()  GameObject.Destroy
.Prefab：AssetBundle.Load    GameObject.DestoryImmidately(true)
.AssetBundle : LoadFromFile LoadFromFileAsync   AssetBundle.Unload(true)

对于小游戏而言总的资源量很少，他们不卸载AB，不卸载AB中的相应的Asset，内存占用也不会超出250MB,所以对于上线的内存要求好像都能满足，只需要在场景切换的时候，对所有的除了切换场景中用到的AB外，都进行Unload,最后调用一个Resource.UnloadAllUseless();一切都是那么的完美。所有这些游戏只需要担心游戏中的卡断问题。

但是如果你的游戏是一个大型游戏，场景非常的大，资源全部加载进来可能超过1G,这个时候，如果不进行地图资源动态的加载卸载，内存肯定通不过上线要求。

存在的问题：
>1.AB卸载错误导致的资源丢失
2.AB或Asset多次加载导致的同一份资源在内存中有多分的问题
3.加载抖动(即刚卸载完，接下来马上又需要进行加载)
4.缓存无法自动的缩减

解决方案：
1：使用GameObject 引用计数
2：使用Cache，但是需要小心处理异步和同步加载同一个AB&Asset的关系的处理
3：使用ObjectPool
4：对Cache,ObjectPool使用Lease模式进行容量的缩减。


这套解决方案的优缺点：
优点：
>sdf
sdf

缺点:
>sdfsf
sdf

sdgfsad


现在有很多的团队的实现方案是：
>1.AB&Fab仅在场景切换的时候进行卸载。（全部只加载，不卸载）
2.AB通过引用计数的方式，进行卸载，但是调用的是AssetBundle.Unload(false)
3.不仅AB通过引用计数的方式进行标记，同时GO也通过引用计数的方式进行卸载。


如果对于上面所讲的
>ObjectPool
Cache
Lease
引用计数


模式不熟悉的同学可以上网搜索，也可以看下面这本书。我觉得讲的非常好。
[向模式的软件架构，卷3 : 资源管理模式][1]
整套经过实战验证的资源管理框架,可以在这里下载
[资源管理框架源码][2]


[游戏优化教程][3]



**4.比较zVal,选择最终需要渲染结果**  
有两种可能，一种是我们是将Unity渲染的场景作为主场景，RayMarching只是当成是一种PostProcess shader，这时候,我们不将RayMarching 的背景色写入ColorBuffer，使用MergeRayMarchingIntoUnity,例如[Fog场景][4]  
```c
#pragma vertex vert   
#pragma fragment frag  
#include "ShaderLibs/Framework3D.cginc"
float4 ProcessRayMarch(float2 uv,float3 ro,float3 rd,inout float sceneDep,float4 sceneCol)  {
 ...//你自己的渲染代码
}
```
效果:  

<p align="center">
<img src="https://github.com/JiepengTan/JiepengTan.github.io/blob/master/assets/img/blog/Show/MergeRaymarchExample.gif?raw=true" width="660"></p>  



  [1]: https://book.douban.com/subject/24527611/
  [2]: https://jiepengtan.github.io/