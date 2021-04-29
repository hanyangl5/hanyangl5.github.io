# Cascade Shadow Maps
在games202中，闫老师介绍了shadowmap，以及用pcf以及pcss做软阴影

shadowmap
![shadow map](figs/SM_xxx.png)
pcf
![pcf](figs/PCF_xxx.png)
pcss
![pcss](figs/PCSS_xxx.png)

在游戏引擎中，更常用的技术是cascaded shadow maps，Nvidia放出的这篇论文中较为详细地介绍了csm的算法

原文地址：https://developer.download.nvidia.com/SDK/10.5/opengl/src/cascaded_shadow_maps/doc/cascaded_shadow_maps.pdf

*图1*

![](figs/p3-11.png)

仅用一张shadowmap记录所有深度信息会有精确度的问题，距离较近的物体没问题，而距离远的物体在shadowmap上占像素很少，CSM为了解决这个问题，把光源空间的视锥体light frustum，分成多个小的视锥体，每个视锥体都计算一个shadowmap

**算法流程如下：**

- For every light’s frustum, render the scene depth from the lights point of view. 
- Render the scene from the camera’s point of view. Depending on the 
fragment’s z-value, pick an appropriate shadow map to the lookup into. 

**Details** 

建议用一个texture array存储shadowmaps

**Shadow-map generation**


![](figs/p3-90.png)

计算eye space下split后的frustum的z值，ds是一个像素在shadowmap上的长度，阴影长度为dp


得出下面等式：

$\frac{dp}{dy}=\frac{n}{z}$

$\frac{dz}{\cos\theta}=\frac{dy}{\cos\phi}$

$\frac{dp}{dz}=\frac{n\cos\phi}{z\cos\theta}$

$\frac{dp}{ds}=\frac{dz\ n\cos\phi}{ds\ z\cos\theta}$

论文[1]中作者测试了不同split方法，包括logarithmic split scheme, uniform split scheme and practical split scheme

![](figs/split.png)

**logarithmic split scheme**

若shadowmap上不同位置的aliasing程度相同，那$\frac{dp}{ds}$应为常数，$\rho=\frac{dz}{z\ ds}$是不变的

可得
$s=\int_0^s\ ds=\frac{1}{rho}\int_n^z\frac{dz}{z}=\frac{1}{\rho}\ln(\frac{z}{n})\ s \in [0,1]$

假设所有的frustum完全覆盖了depth range

$\rho=\ln(\frac{f}{n})$

在depth范围内积分 

$s=\frac{\ln(z/n)}{\ln(f/n)}$

**Final scene rendering**

在ps阶段，每个像素的深度值（projection后的）和不同shadowmap的z-range比较，找到对应的第ith级shadowmap，再转换到worldspace并乘上对应的light mat到光源空间，变换为0-1的texcoord，就可以查shadowmap上的值计算阴影了

**对比**

用4 split csm渲染的结果

![](figs/p11-40.png)

![](figs/p11-41.png)

1 split

![](figs/p12-45.png)

3 split，效果比1 split好很多

![](figs/p12-46.png)

[1]Parallel-split shadow maps for large-scale virtual environments
