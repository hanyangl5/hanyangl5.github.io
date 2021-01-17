2020.7-8写的软渲染器notes

# C++ Soft Renderer


# *Intro*
自己看图形学也大概有一年了,从一开始看opengl，raytracing，这学期又跟了一门cg入门的课程，打算写一个光栅化软渲染器来整合一下自己所学的东西，复习一下所学的知识。

# *3D数学*（右手坐标系）

<!-- 图形学中的向量一般是4维的，矩阵一般是4×4的，我们定义```Vector4f```，```Matrix4f```来表示这两种数据结构 -->

## 变换矩阵

### 平移

平移操作将空间中的一个点 $ p(p_x,p_y,p_z,1) $ 沿 $ t(t_x,t_y,t_z) $ 方向移动一段距离，需将点p左乘变换矩阵 $ T(t) $ ,
 $ \\T(t)=\begin{bmatrix}1 & 0 & 0 & t_x\\0&1&0&t_y\\0&0&1&t_z\\0&0&0&1\\\end{bmatrix} $ 

平移后点p的坐标为 $ p'=(p_x+t_x, p_y +t_y, p_z +t_z, 1) $ 


### 旋转

**二维旋转**

**绕原点的二维旋转**

我们要将p绕原点旋转 $ \phi $ 到点 $ q $ ,即需要一个矩阵R，使得

 $ q=R\ p \\
\begin{bmatrix}rcos(\theta+\phi)\\ 
rsin(\theta+\phi)\end{bmatrix}=R\begin{bmatrix}rcos\theta\\ 
rsin\theta\end{bmatrix}\\ $ 
 $ 
\begin{bmatrix}rcos\theta cos\phi-rsin\theta sin\phi\\
rsin\theta cos\phi+rcos\theta sin\phi\end{bmatrix}=R\begin{bmatrix} rcos\theta\\ 
rsin\theta\end{bmatrix}\\ $ 
解得
 $ R=\begin{bmatrix}
cos\phi & -sin\phi\\ 
sin\phi& cos\phi
\end{bmatrix} $ 

**绕任意点的二维旋转**

1. 将旋转点x移动到原点
2. 绕原点进行二维旋转
3. 将x移动回原来的位置

### 三维旋转

空间直角坐标系通常有两种表示方式，左手坐标系和右手坐标系，也是DirectX和OpenGL中不同的地方，本文中的坐标系均以右手坐标系为例

#### 绕轴的旋转

- 绕z轴旋转

<img src="https://pic.downk.cc/item/5f03669014195aa5942e4945.jpg" height="200" width="200">

 $ q=R_z*p\\ $ 
 $ 
\begin{bmatrix}
rcos(\theta+\phi)\\ 
rsin(\theta+\phi)\\ 0
\end{bmatrix}
=R_z
\begin{bmatrix}rcos\theta\\ 
rsin\theta\\ 0\end{bmatrix}\\ $ 
 $ 
\begin{bmatrix}
rcos\theta cos\phi-rsin\theta sin\phi\\
rsin\theta cos\phi+rcos\theta sin\phi\\0\\
\end{bmatrix}
=R_z
\begin{bmatrix} rcos\theta\\
rsin\theta\\0 \\
\end{bmatrix}\\ $ 
解得
 $ R_z=\begin{bmatrix}
cos\phi & -sin\phi&0\\ 
sin\phi& cos\phi&0\\
0&0&1
\end{bmatrix} $ 

- 绕x轴旋转

<img src="https://pic.downk.cc/item/5f03669014195aa5942e493b.jpg" height="200" width="200">

 $ q=R_x*p\\ $ 
 $ 
\begin{bmatrix}
0\\
rcos(\theta+\phi)\\ 
rsin(\theta+\phi)
\end{bmatrix}
=M
\begin{bmatrix}
0\\
rcos\theta\\ 
rsin\theta
\end{bmatrix}\\ $ 
 $ 
\begin{bmatrix}
0\\rcos\theta cos\phi-rsin\theta sin\phi\\
rsin\theta cos\phi+rcos\theta sin\phi
\end{bmatrix}
=M\begin{bmatrix} 0\\rcos\theta\\ 
rsin\theta\end{bmatrix} $ 

解得
 $ R_x=\begin{bmatrix}
1&0&0\\
0&cos\phi&-sin\phi\\
0&sin\phi&cos\phi
\end{bmatrix} $ 


- 绕y轴旋转

<img src="https://pic.downk.cc/item/5f03669014195aa5942e4940.jpg" height="200" width="200">

 $ q=R_y*p \\ $ 
 $ 
\begin{bmatrix}
rsin(\theta+\phi)\\0\\rcos(\theta+\phi)
\end{bmatrix}
=R_y
\begin{bmatrix}
rsin\theta\\0\\rcos\theta
\end{bmatrix}\\ $ 
 $ \begin{bmatrix}
rsin\theta cos\phi+rcos\theta sin\phi\\0\\rcos\theta cos\phi-rsin\theta sin\phi
\end{bmatrix}
=R_y
\begin{bmatrix}
rsin\theta\\0\\rcos\theta
\end{bmatrix} $ 

解得
 $ R_y=\begin{bmatrix}
cos\phi&0&sin\phi\\
0&1&0\\
-sin\phi&0&cos\phi
\end{bmatrix} $ 

将上述三个旋转矩阵 $ R_x,R_y,R_z $ 组合起来
 $ R_{x,y,z}(\alpha,\beta,\gamma)=R_x(\alpha)R_y(\beta)R_z(\gamma) $ 

这样的旋转也叫Euler旋转

#### 沿任意轴的旋转/Rodrigues’ Rotation Formula

和二维绕任意点旋转类似，我们将旋转轴 $  n $ 旋转至与坐标轴重合（以 $  x $ 轴为例），然后绕 $  x $ 轴旋转 $ {\alpha} $ ，再旋转轴旋转回原来的位置

<!-- <img src="https://pic.downk.cc/item/5f0aebed14195aa5941fe522.jpg" widht="200" height="200"> -->

整个流程如下

 $ q=M^TR_x(\alpha)Mp\\ $ 
 $ cos\theta=\frac{z}{\sqrt{y^2+z^2}}\\
sin\theta=\frac{y}{\sqrt{y^2+z^2}}\\ $ 
 $ R_x(\theta)=\begin{bmatrix}
1&0&0&0\\
0&\frac{z}{\sqrt{y^2+z^2}}&-\frac{y}{\sqrt{y^2+z^2}}&0\\
0&\frac{y}{\sqrt{y^2+z^2}}&\frac{z}{\sqrt{y^2+z^2}}&0\\
0&0&0&1
\end{bmatrix}\\ $ 
 $ cos\phi=\frac{x}{\sqrt{x^2+y^2+z^2}} \\ 
sin\phi=\frac{\sqrt{y^2+z^2}}{\sqrt{x^2+y^2+z^2}}\\ $ 
 $ R_y(\phi)=\begin{bmatrix}
\frac{x}{\sqrt{x^2+y^2+z^2}}&0&\frac{\sqrt{y^2+z^2}}{\sqrt{x^2+y^2+z^2}}&0\\
0&1&0&0\\
-\frac{\sqrt{y^2+z^2}}{\sqrt{x^2+y^2+z^2}}&0&\frac{x}{\sqrt{x^2+y^2+z^2}}&0\\
0&0&0&1
\end{bmatrix}\\ $ 
 $ R=R_x(-\theta)R_y(-\phi)R_x(\alpha)R_y(\phi)R_x(\theta) $ 

最终旋转矩阵
 $ R=\begin{bmatrix}
  cos\alpha+(1-cos\alpha)n_x^2&(1-cos\alpha)n_xn_y-n_zsin\alpha&(1-cos\alpha)n_xn_y+n_ysin\alpha\\ 
(1-cos\alpha)n_xn_y+n_zsin\alpha&cos\alpha+(1-cos\alpha)n_y^2&(1-cos\alpha)n_yn_z-nx_sin\alpha\\
(1-cos\alpha)n_xn_z-n_ysin\alpha&(1-cos\alpha)n_yn_z+n_xsin\alpha&cos\alpha+(1-cos\alpha)n_z^2
\end{bmatrix}\\
=cos(\alpha)I+(1-cos\alpha)nn^T+sin\alpha
 $ 

*欧拉旋转的问题：roll轴与up向量重合时,绕roll的旋转会失效，cross(roll,up)无意义*

<!-- ####  四元数旋转

见 []()
 -->

### 缩放

缩放矩阵的形式如下，可以将某个点/向量沿 $ x,y,z $ 轴分别缩放 $ s_x,s_y,s_z $ 倍

   $ S(s)=\begin{bmatrix}
  s_x & 0 & 0 & 0\\
  0&s_y&0&0\\
  0&0&s_z&0\\
  0&0&0&1\\
  \end{bmatrix} $ 


# *几何变换*

Rendering Pipeline的几何阶段包含以下几个部分

![](https://pic.downk.cc/item/5f0407f214195aa5946cc07c.png)

## 模型坐标变换

我们所建立的坐标系是世界坐标系，渲染的物体在世界坐标系中的位置是要人为规定的，每个模型都有自己独立的坐标系，第一步就是将物体在模型坐标系的坐标转换成世界坐标系的坐标

可以用将点左乘平移和旋转矩阵来达到目的

平移和旋转的次序不同，变换的结果也不同（矩阵乘法运算无交换律）

**Model矩阵**
 $ \\M=R(\alpha) T(t) $ 


## 视图变换

我们只需要渲染在摄像机视野范围内的物体

我们希望  $ \vec{eyepos}-\vec {at} $ 指向 $ z $ 轴负方向，即相机朝向的方向是屏幕里侧


![](https://www.realtimerendering.com/figures/RTR4.02.04.png)

**构建View矩阵**

已知 $ eyepos,at,up $ 信息
根据空间几何的知识可以推算出视图坐标系的坐标基
 $ \\zaxis=eyepos-at\\ 
xaxis=up\times zaxis\\ 
yaxis=zaxis\times xaxis $ 
 $ \\V=\begin{bmatrix}
xaxis_x&xaxis_y&xaxis_z&0\\
yaxis_x&yaxis_y&yaxis_z&0\\
zaxis_x&zaxis_y&zaxis_z&0\\
0&0&0&1
\end{bmatrix} $ 
将摄像机平移到原点
 $ \\V=\begin{bmatrix}
xaxis_x&xaxis_y&xaxis_z&-dot(xaxis,eyepos)\\
yaxis_x&yaxis_y&yaxis_z&-dot(yaxis,eyepos)\\
zaxis_x&zaxis_y&zaxis_z&-dot(zaxis,eyepos)\\
0&0&0&1
\end{bmatrix} $ 

## 投影变换


投影变换我们仅考虑透视投影，我们的摄像机所看到的范围是一个视锥体

<!-- ，我们希望把视锥体内部的物体映射到齐次裁剪空间内，显示再屏幕上，视锥体外的物体剔除掉 -->
<!-- 
<!-- ![](https://www.realtimerendering.com/figures/thumb/RTR4.02.05.jpg) -->
<!-- 
- 透视投影
 $ \\P_{perspective}=\begin{bmatrix}
\frac{z_{near}}{n_{w}}&0&0&0\\
0&\frac{z_{far}}{n_{h}}&0&0&\\
0&0&\frac{-(z_{near}+z_{far})}{z_{far}-z_{near}}&\frac{-2z_{near}z_{far}}{z_{far}-z_{near}}\\
0&0&-1&0
\end{bmatrix} $ 

- 正交投影
 $ \\P_{orthographic }\begin{bmatrix}
  \frac{1}{w}&0&0&0\\
  0&\frac{1}{h}&0&0\\
  0&0&\frac{-2}{z_{far}-z_{near}}& -\frac{z_{far}+z_{near}}{z_{far}-z_{near}}\\
  0&0&0&1
\end{bmatrix} $ 

在游戏中，我们常见的调整视野的方式有调节fov（field of view），分辨率（width，height），在构造投影矩阵也应使用这些参数 -->


设投影变换前的点 $ p(px,py,pz) $ ，第一步我们将视锥体内的点映射到近平面，根据相似三角形得到下面两个等式(z坐标均为负值)
 $ \frac{px}{pz}=\frac{px'}{z_{near}}\\
\frac{py}{pz}=\frac{py'}{z_{near}}\\ $ 


将矩形内的x，y映射成[-1,1]
 $ \frac{py'}{H}=\frac{py''}{2}\\
py''=\frac{py}{pz}\frac{1}{tan(\frac{fov}{2})}\\ $ 
 $ \frac{px'}{H·aspect\_ratio}=\frac{px''}{2}\\
px''=\frac{px}{pz}\frac{1}{tan(\frac{fov}{2})·aspect\_ratio} $ 

这样我们就获得了变换后的x，y坐标，也就得到透视矩阵的前两行
我们希望w保存z坐标，即w=-z
 $ \\\begin{bmatrix}
  \frac{1}{tan(\frac{fov}{2})·aspect\_ratio}&0&0&0\\
  0&\frac{1}{tan(\frac{fov}{2})}&0&0\\
  .&.&.&.\\
  .&.&-1&.
\end{bmatrix} $ 

当Frustum内的点投影到近剪裁平面的时候，所有位于线段p'p上的点，最终都会投影到p'点。

在现代图形API中，为了方便光栅化阶段对屏幕空间的坐标插值，将z映射为1/z的非线性形式，将z''写成1/z的一次表达式形式，如下
 $ \\z''=\frac{a\ pz+b}{-pz} $ 

将z端点带入，近平面z=near时为-1，z=far时为1

解得
 $ \\a=\frac{-(z_{near}+z_{far})}{z_{far}-z_{near}}\\
b=\frac{-2z_{near}z_{far}}{z_{far}-z_{near}} $ 

最终的透视投影矩阵
 $ \\P_{perspective}=\begin{bmatrix}
\frac{1}{aspect\_ratio·tan(\frac{fov}{2})}&0&0&0\\
0&\frac{1}{tan(\frac{fov}{2})}&0&0&\\
0&0&\frac{-(z_{near}+z_{far})}{z_{far}-z_{near}}&\frac{-2z_{near}z_{far}}{z_{far}-z_{near}}\\
0&0&-1&0
\end{bmatrix} $ 

## 裁剪

裁剪这一步的作用是提高渲染效率，剔除裁剪空间外部的图形，如果空间边缘分割了某个图形，将其分裂成数个小三角形再进行下一步渲染

![](https://www.realtimerendering.com/figures/thumb/RTR4.02.06.jpg)


## 透视除法

**齐次坐标的意义**

在欧几里得空间内，两条平行的线是永远无法相交的

![](http://www.songho.ca/math/homogeneous/files/railroad.jpg)

然而，在投影空间中，情况就不再如此，例如，当火车铁路远离眼睛时，上图的铁路变得越来越窄。最后，两个平行导轨在地平线上相遇，这个点的z值是无穷

将3D裁剪坐标系下的点除以w从3D裁剪坐标(x,y,w)转换为2D坐标

$ \\
\begin{bmatrix}x\\
y\\z\\w\\
 \end{bmatrix}  
=\begin{bmatrix} 
\frac{x}{z}\\
\frac{y}{z}\\
\frac{z}{w}\\
1\\
\end{bmatrix} $

## 屏幕映射

![](https://www.realtimerendering.com/figures/thumb/RTR4.02.07.jpg)


将区间为[-1,1]的x，y坐标映射到屏幕空间上
 $ \\x=\frac{1}{2}width\cdot(x+1)\\
y=\frac{1}{2}height\cdot(y+1) $ 


# *光栅化*

## 直线绘制算法

### bresenham直线生成算法


bresenham是一种高效的直线绘制的算法，也是最常用的直线绘制算法

给定两个点 A（x1、y1）和 B（x2、y2）的坐标。在像素的计算机屏幕上查找绘制线 AB 所需的所有中间点


算法步骤伪代码
```
dx = abs(x2 - x1), dy = abs(y2 - y1)
m = dy / dx //计算斜率
putpixel(x, y)// ouput first pixel
if  m > 1
     pk = (2dy - dx)
     for  x = x1; x < x2; x++
        if  pk < 0
           pk = pk + 2dy
        if  pk > 0
           pk = pk + 2dy − 2dx
           y++
        putpixel(x,  y)
if m < 1
    pk = (2dx - dy)
    for\  y = y1; y < y2; y++
        if  pk < 0
          pk = pk + 2dx
        if  pk > 0
           pk = pk + 2dx − 2dy
           x++
        putpixel(x, y)
```

直线生成算法用于绘制线框图

<img src="https://pic.downk.cc/item/5f057f5714195aa594029c67.png" height="200" width="300">


## 光栅化

光栅化阶段，遍历三角形boundingbox内像素，判断像素是否在三角形内

### 扫描线算法

规定三角形的三个顶点v1，v2，v3的y值y1<y2<y3，我们将三角形分解成上下两部分，根据插值计算出新的点v4

<img src="http://www.sunshine2k.de/coding/java/TriangleRasterization/generalTriangle.png" height="200" width="400">
<img src="https://pic2.zhimg.com/80/v2-8f782cb6db112478d4cf86122abdcba0_720w.jpg" height="200" width="200">

以三角形的长边为底，一行一行向上/向下绘制像素点。

缺点：插值的精度损失导致的多个三角形间存在接缝的问题

### 重心坐标

已知三角形的三个顶点 $ v1，v2，v3 $ 

用  $ \alpha ·v1+\beta·v2+\gamma·v3,\alpha+\beta+\gamma=1 $  可以表示三角形平面上任意点
如果做约束 $ \alpha,\beta,\gamma\in[0,1] $ ,上式可以表示三角形内部的任意点

反过来，如果给定任意点 $ (x,y) $ ，我们可以写出它的重心坐标


 $ (x,y)=\alpha\ v1+\beta\ v2+\gamma\ v3\\
\alpha+\beta+\gamma=1\\
x=\alpha\ v1.x+\beta\ v2.x+\gamma\ v3.x\\
=\alpha\ v1.x+\beta\ v2.x+(1-\alpha-\beta)\ v3.x\\
y=\alpha\ v1.y+\beta\ v2.y+(1-\alpha-\beta)\ v3.y\\
\beta=\frac{\alpha(v3.x-v1.x)+(x-v3.x)}{v2.x-v3.x}=\frac{\alpha(v3.y-v1.y)+(y-v3.y)}{v2.y-v3.y}\\
\alpha=\frac{(y-v3.y)(v2.x-v3.x)-(x-v3.x)(v2.y-v3.y)}{(v3.x-v1.x)(v2.y-v3.y)-(v3.y-v1.y)(v2.x-v3.x)}\\
\beta=\frac{(x-v3.x)(v2.y-v3.y)-(y-v3.y)(v2.x-v3.x)}{(v3.y-v1.y)(v2.x-v3.x)-(v3.x-v1.x)(v2.y-v3.y)}\\
\gamma=1-\alpha-\beta $ 


如果重心坐标 $ \alpha,\beta,\gamma $ 满足 $ \alpha,\beta,\gamma\in[0,1] $ 那么就可以判断点在三角形内




<!-- ### edgefunction做法
$e_{x,y}=n((x,y)-p)=0$
n是edge normal，p是边上一点 

$e_{x,y}>0,inside\\ 
e_{x,y}<=,outside\\
e_{x,y}=0,on the line$  -->

### 深度测试

经过光栅化的过程，我们已经可以绘制出基本的图形了，但仍有一个问题需要解决，后绘制的图形总是在先绘制的图形前面，也就是我们总能看到最后绘制的图形，这与现实是不相符的，我们用一个数组记录屏幕上每个像素的深度值

由点的重心坐标，对三角形三个点的z值插值
 $ zp=\alpha\ v1.z+\beta\ v2.z+\gamma\ v3.z $ 

如果 $ zp $ 值小于深度缓冲中相应位置存储的 $ z $ 值，将 $ zp $ 写入深度缓冲，并绘制出像素的颜色



<!-- # *纹理映射*

通过stb_image库读取纹理，模型文件中的(u,v)坐标采样纹理

我们把它应用到渲染器中，以一个含4个顶点的正方形为例，顶点坐标和纹理坐标都是(0,0)，(0,1)，(1,0)，(1,1)，并贴一张木箱贴图

<img src="https://learnopengl.com/img/textures/container.jpg" height="200" width="200">


逻辑上来说，木箱上同一个点的uv坐标是不变的，与视角，物体变换无关

但是，我们在直接插值uv坐标后发现并不正确
<img src="https://img.imgdb.cn/item/6002dc6a3ffa7d37b3fcc367.png" height="200" width="320">


先来解释一下为什么会出现这样的问题，下图是上面正方形的图例，红点是正方形的中心，uv由光栅化阶段的插值计算得到，为(0.5,0.5)，正好采样木箱的正中心


<img src="https://img.imgdb.cn/item/6002de2b3ffa7d37b3fda0fe.png" height="200" width="200">

这个正方形经过旋转变成下图所示的形状，红点依然是正方形的中心，但uv为(0.5,0.5)的点变成黄点了（仅作参考，与三角形索引顺序有关），黄点采样木箱的正中心，这显然不正确

<img src="https://img.imgdb.cn/item/6002de2b3ffa7d37b3fda101.png" height="200" width="140">

不仅是uv，顶点，法线也都会有问题

**解决方法**

$(e_x,e_y,e_z)$ 代表某一点view space的坐标，这一点由 $\alpha v_1+\beta v_2+\gamma v_3$ 计算得到

在光栅化阶段，插值是在screen space下计算，

$(\frac{e_x}{w},\frac{e_y}{w},\frac{e_z}{w})= \alpha\frac{v_1}{w}+\beta\frac{v_2}{w}+\gamma\frac{v_3}{w}$ -->


<!-- 
透视纹理矫正
![](figures/texture_correction.png)



还记得吗，我们在投影变换中把z值映射成了关于1/z的函数

$z''=\frac{a}{pz}+b$
$\\a=\frac{-(z_{near}+z_{far})}{z_{far}-z_{near}}\\
b=\frac{-2z_{near}z_{far}}{z_{far}-z_{near}}$

在纹理映射时，如果我们用z值进行插值，则会出现如上图所示的问题，


我们先做透视除法再插值
$f_x=\alpha\ \frac{x_0}{\omega_0}+(1-\alpha)\ \frac{x_1}{\omega_1}$
我们希望先插值再做透视除法
$f_x=\frac{\beta\  x_0+(1-\beta)\ x_1}{\beta\ \omega_0+(1-\beta)\ \omega_0}$


$\beta=\frac{\alpha\ \omega_0}{(1-\alpha)\ \omega_1+\alpha\ \omega_0}$


最近在做作业时遇到了一些以前没接触过的东西，

法线贴图normal mapping，也叫凹凸贴图bump mapping，是指对模型表面应用一层纹理，从而改变表面的法线方向，进而影响光照
下图是一张砖块法线贴图

![](normalmap.png)

在法线贴图中，我们用颜色的r，g，b值来存存储法线向量的x，y，z值
贴图颜色整体偏蓝色的原因是法线方向都在轴(0,0,1)附近
我们希望法线都是指向z轴的，如果模型表面是扭曲的，或者某个点的法线方向不是z轴，而是指向其他方向，如(1,0,0)，那么法线贴图就不能直接贴在模型上，因此，我们要将法线贴图中的法线方向转换到模型空间


我们称法线贴图所在的空间为切线空间，我们对模型上的每个点计算一个TBN矩阵，使得从贴图中获取法线方向垂直于该点，要构建这样的空间，我们需要三个相互垂直的向量t，b，n。

首先我们获得该点的法线n，切线t
n cross t得到向量b

构成的矩阵如下
$\begin{bmatrix}
t_x&t_y&t_z&0\\
b_x&b_y&b_z&0\\
n_x&n_y&n_z&0\\
0  &  0  &0&1
\end{bmatrix}$
通过这个矩阵可以将向量从模型空间转换到切线空间
<!-->
在实际情况中，为了节省内存，我们只存贮t，b 用tcrossb计算n的方向
<!-->
 -->

# Conclusion

渲染器还有很多不完善的地方，渲染效率较低，着色部分还没写，以后找时间填坑吧

### reference:

https://learnopengl-cn.github.io/
https://blog.csdn.net/qq_28773183/article/details/80083607
https://blog.csdn.net/csxiaoshui/article/details/65446125
https://www.cnblogs.com/wtyuan/p/12324495.html
https://www.zhihu.com/question/40624282
https://learnopengl-cn.readthedocs.io/zh/latest/04%20Advanced%20OpenGL/01%20Depth%20testing/
https://www.comp.nus.edu.sg/~lowkl/publications/lowk_persp_interp_techrep.pdf
https://yangwc.com/2019/05/02/SoftRenderer-3DPipeline/
《Real-Time Rendering Fourth Edition》