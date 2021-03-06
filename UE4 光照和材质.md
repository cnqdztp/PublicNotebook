# 灯光系统





## 理解灯光概念

对于灯光方案非常关键

实时渲染本质是CG模拟物理世界，光和影可以用多种手段分别模拟

光照和材质相互影响，相关多种因素共同构成最终画面

观众的主观感受是最根本的美术标准

光照复杂度带来计算成本，需要根据目标设备的算力灵活取舍

## 灯光类型

**点光源：**

​	通过一个点向周围360°的球形空间发射无方向性的灯光

**聚光源：**

​	投射带有方向性的锥形光源

**定向光源：**

​	全局影响，类似于太阳；通过启用	影响场景选项来影响整个关卡场景。

**矩形光源：**

​	以平面发射光源。模拟窗户照进室内；模拟显示器；摄影棚的聚光板。

**天光：**

​	全局影响，天光抓取整个环境的颜色，叠加到天光上。



### 通用参数

动态阴影：

​	氛围光、装饰光可以不开启，以便节省计算资源

可视：

​	渲染/不渲染

Lightmass：

​	很重要

灯光函数：

Light Profile：

​	赋予灯光特殊的形状效果



###  专有参数

**定向光源参数**

光束：

​	模拟太阳的光源，使得光源可以直接可见

![屏幕截图 2021-03-21 212430](D:\MyDocument\GitHub\PublicNotebook\Assests\屏幕截图 2021-03-21 212430.png)

级联阴影贴图：

​	让相机远离阴影的时候，降低阴影的质量



**天光**

实时捕获：

​	（保证天光的移动性是可移动）

​	·*启用*之后，大气雾，指数云以及天空材质都会被计算进天光的颜色计算当中

​	·*源类型*可以指定捕获场景、或者指定一个立方贴图。

​	·*天空距离阈值*可以指定捕获多少距离内的环境

​	·*强度*可以指定天光的亮度；*颜色*可以指定天光的颜色；

​	

距离场环境遮蔽：

​	计算模型接触的范围，在范围内模拟阴影

重捕获：

​	按照参数重新抓取环境信息



## 直接光照和间接光照

·不同于离线渲染，实时渲染需要在各方面考虑效果和效率的平衡

·直接光照指光源到材质表面**首次反弹**形成的光照效果，效率高，效果差

·间接光照指直接光照反弹后在其它材质表面**二次或多次反弹**形成的光照效果，计算量较大，需要通过多种手段模拟，但同时也会产生更美观的效果![屏幕截图 2021-03-21 214831](D:\MyDocument\GitHub\PublicNotebook\Assests\屏幕截图 2021-03-21 214831.png)



默认来说，只有直接光照。

**添加间接光照**

​	·直接在应该补光的地方增加点光源。-->主光源的参数不会被更新到补光上

​	·使用Lightmass重要体积（Actor-->体积）。

> The **Lightmass Importance Volume** controls the area that **Lightmass** emits photons in, allowing you to concentrate it only on the area that needs detailed indirect lighting. Areas outside the **importance volume** get only one bounce of indirect lighting at a lower quality.

​	·使用天光，移动性改为可移动；确保环境中有可以捕获的环境信息。（注意！没有模拟多次反弹，而是将立方体贴图直接渲染在材质上）

​	·打开SSGI（噪点、无法反射屏幕外的像素）



## 反射和环境

·反射属于间接光照

·UE中模拟反射存在多种方式

·高光不等于反射、属于直接光照



## 全局照明

·3D全局照明模拟现实世界光线多次反弹形成的光照效果

·全局照明=直接光照+间接光照

·全局照明的实现方式多种多样效果好、效率高、成本低的全局照明方案是我们持续追求的目标

·虚幻引擎现有G方案：Lightmass、天光模拟、 ssG|、RTXG、 Lumen（UE5）



## 灯光移动性（重要）

·灯光移动性灯光移动性极大地影响到不同类型的灯光对静态或动态光照方案的照明方式

·静态仅支持**静态**光照方案（烘焙光照贴图）

·固定支持**静态**和**动态**光照方案（光照贴图+体积光照贴图、静态反射、直接光照、实时阴影）

·动态仅支持**动态**光照方案（直接光照+实时阴影）

·灯光移动性和物体移动性互相影响



[静态](https://docs.unrealengine.com/zh-CN/BuildingWorlds/LightingAndShadows/LightMobility/StaticLights/index.html)

[固定](https://docs.unrealengine.com/zh-CN/BuildingWorlds/LightingAndShadows/LightMobility/StationaryLights/index.html)

[可移动](https://docs.unrealengine.com/zh-CN/BuildingWorlds/LightingAndShadows/LightMobility/DynamicLights/index.html)



> 理解：烘焙光照贴图、体积光照贴图、静态反射、直接光照、实时阴影
>
> 

## 选择光照方案



### 静态光照方案

·静态光照方案基于预计算的光照方案，主要指 LightMap
·细节效果好，运行时性能高，实时灵活度低，对资产制作要求高
·UE光照贴图烘焙工具 Lightmass
·Lightmass设置（全局、灯光、物体）
·GPU Lightmass通常用于室内封闭环境



### 动态光照方案

·实时计算的直接光照、间接光照、反射、环境光遮蔽

·实时性能消耗高、效果精度低、灵活度高、对资产制作要求低

·全动态的直接光照、反射、投影

·通常用于室外开放环境



## 阴影

UE4采用多种手段模拟阴影





> 学习灯光参数非常重要，各类参数适用于不同的光照方案
>
> 提供众多参数是为了提供效果、效率和制作成本的平衡
>
> 不需要背参数，但是要理解参数调整的目的



## 灯光设计

**设计原则**

1. 按照**自然规律**布光，避免无意义的补光。氛围灯除外。
2. 灯光设置追求**视觉美观**和统一可控，而非绝对的物理真实
3. 优秀的灯光设计需要综合考虑**光照、后处理、雾效、材质、粒子效果**等多种因素

**优化原则**

1. 根据灯光方案合理布光，合理选择**灯光特性和精度**
2. 优化思路：**实时计算越少，性能越高**
3. 预计算光照不仅能提高**性能**，而且能提高**效果**，但是可能会带来**制作成本**的显著提高
4. 项目实战中，并不是性能越高越好，依然需要考虑效果、性能、制作成本三者之间的平衡



# 材质系统



## 基础概念

·材质和shader的关系

> 本质来说，材质就是一段shader代码

·贴图和材质的关系

·材质和灯光的关系

## PBR材质及工作流

PBR：基于物理的渲染；虚幻引擎的渲染默认基于PBR

 

## 贴图概念和贴图编辑器

- 贴图本质上就是用像素颜色和位置存储的数据

- 通常通过材质节点采样贴图中的信息
- 贴图原始数据导入UE的时候可以根据需要进行转换和压缩
- UE支持的常见贴图格式
- 贴图制作方式简述

 

贴图尺寸的规律：长宽应该是2的指数，比如256或者1024，长宽数值不一定一直；好处：方便贴图压缩；生成贴图Mip；

> 贴图编辑器中的细节部分一般来说不需要修改，UE会自动识别并填写信息

## 材质概念和材质实例编辑器

5.42:00~

## 其他材质相关资产类型



## 材质案例简析



## 材质性能优化



## 材质编辑器的更多细节



## 材质框架



## 分层材质系统

