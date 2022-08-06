---
title: "计算机图形学基础学习笔记-光线追踪"
date: 2022-07-28T17:36:32+08:00
draft: false
tags: [图形学]
categories: 图形学
mathjax: true
markup: pandoc
---

## 为什么需要光线追踪

光栅化不能很好地处理全局效果。例如软阴影，以及光照的多次反射。

虽然光栅化很快，但是质量不非常好、不真实。

相比之下光线追踪效果会好很多，但是也非常慢。在过去算力不发达的时候，光栅化是一个实时的算法，而光线追踪是一个离线的算法。

## 基础的光线追踪算法

### 光线投射

1. 对每一个像素投射一条光线来生成一张图
2. 对光源发射一条光线来检查阴影

**Pinhole Camera Model**

首先从眼睛，或者说相机发出一条光线，穿过图像平面上的一个像素。然后和空间中的一个物体边缘相交。当然可以和一个物体多次、或者和多个物体相交，我们需要的是最近的。

然后对最近的这个交点进行着色计算。例如使用Blinn-Phong模型。

**Recursive(Whitted-Style) Ray Tracing**

仍然从相机发出一条光线，穿过图像平面上的一个像素。然后和空间中的一个物体边缘相交。当然，如果是较为镜面的物体，会继续反射光线的路径和下一个物体边缘相交。直到不出现镜面反射。如果是玻璃等透射物质，则要沿着折射光线继续，直到不透明物体。

### 光线的数学表示

光线由光源点和方向向量表示。

$$
\bm r(t) = \bm o+t\bm d\quad 0\leq t<\infty
$$

其中$t$是时间，$\bm o$是光源向量，$\bm d$是方向向量。

### 光线相交

**对于隐式表示的物体**

判断是否相交，例如一个球面：

$$
(\bm{p}-\bm{c})^2-R^2=0
$$

其中$\bm p$是球面上的点，$\bm c$是球心，和

$$
\bm r(t) = \bm o+t\bm d\quad 0\leq t<\infty
$$

相交，则要满足

$$
(\bm o+t\bm d-\bm c)^2-R^2=0
$$

只有$t$一个未知数，解即可。

其他的隐式表示的，或者说用代数形式表示的物体都可以这么判断相交。

**对于三角形网格**

首先，三角形一定能够确定一个平面。所以先判断光线和平面的相交，以及计算交点。再计算交点是否在三角形内。

一个平面可以表述为上面的一点和平面的法向量。

所有在平面上的点满足如下等式

$$
(\bm p-\bm p')\cdot \bm N = 0
$$

其中，$\bm p'$是平面上已知一点，$\bm N$是平面法向量。

判断光线和平面的交点：

和

$$
\bm r(t) = \bm o+t\bm d\quad 0\leq t<\infty
$$

联立，得

$$
(\bm o+t\bm d-\bm p')\cdot \bm N = 0
$$

解得

$$
t = \frac{(\bm p'-\bm o)\cdot \bm N}{\bm d\cdot\bm N}
$$

应确保$0\leq t<\infty$

*直接计算出交点重心坐标的方法*

Moller Trumbore Algorithm:

$$
\bm O+t\bm D = (1-b_1-b_2)\bm P_0+b_1\bm P_1+b_2\bm P_2
$$

$$
\begin{bmatrix}
t \\
b_1 \\
b_2
\end{bmatrix}=
\frac{1}{\bm S_1\cdot\bm E_1}
\begin{bmatrix}
\bm S_2\cdot\bm E_2 \\
\bm S_1\cdot\bm S \\
\bm S_2\cdot\bm D
\end{bmatrix}
$$

其中：

$$
\bm E_1=\bm P_1 - \bm P_0
$$

$$
\bm E_2=\bm P_2 - \bm P_0
$$

$$
\bm S=\bm O - \bm P_0
$$

$$
\bm S_1=\bm D \times \bm E
_2
$$

$$
\bm S_2=\bm S \times \bm E
_1
$$

## 光线追踪加速

### 包围盒

一个物体被完整地包含在一个长方体中，这个长方体称之为包围盒。有时有多个物体在内。

显然如果光线没有击中包围盒，光线也就不会击中物体。

由于长方体的六个面确定了六个平面，长方体又可以看作是三组平行平面的交叉。

我们通常会使用轴对齐的包围盒。即三组平面都平行于坐标面。

对每一组平面都求光线和其交点，都会求出一个近点和远点。将三组近点和远点比较，求出最远的近点和最近的远点，就是和包围盒的交点。当然同时注意$t\geq 0$

使用轴对齐的包围盒可以减少算法常数

例如对齐x轴时

$$
t = \frac{(\bm p'-\bm o)\cdot \bm N}{\bm d\cdot\bm N}=\frac{\bm p'_x-\bm o_x}{\bm d_x}
$$

### 空间均匀分割

找到一块区域，均匀地划分成正方形（三维情况为正方体）网格。在每个网格记录物体边缘的重叠情况。

对于光线穿过的每个网格中，枚举和网格有交集的所有物体，检测是否和光线相交。

网格划分太少，则很难于无划分区分开来。划分太多，则会有很多和网格的计算。

一个启发式的算法是，网格数等于C乘以物体数。在三维中C为27.

处理较好的情况是，有很多物体的情况。而处理不好的情况是，一个体育馆中心有一个茶壶等类似情况时。

### 不均匀的分割

见数据结构笔记KD-Tree。

下面讲解光线和KD-Tree中的小空间的相交处理。

首先从根节点开始。光线首先会和根节点表述的空间有两个交点。

然后判断和两个子节点是否有相交。如果没有则忽略。如果有，则持续往下分割。直到没有子节点，就判断小空间内的物体是否和光线相交。

### 层次包围盒(BVH)

见数据结构笔记。

## 辐射度量学

### 辐射能量

辐射能量是电磁辐射的能量。以焦耳（J）做单位，$Q$为符号。

### 辐射通量(Radiant Flux)

辐射通量是每单位时间接收、发射、反射、传输的能量。

$$
\Phi\equiv\frac{dQ}{dt}
$$

单位是瓦特（W），或流明（lm）。

但流明和瓦特略有不同，流明作为光通量的单位，只考虑了可见光的通量，而瓦特作为辐射通量考虑了电磁波的全部通量。

### 辐射强度(Radiant Intensity)

辐射强度是每单位立体角所能从点光源接收到的辐射通量。

$$
I(\omega)\equiv\frac{d\Phi}{d\omega}
$$

对于可见光单位是坎德拉（cd），对于一般电磁波是$W/sr$。

### 立体角

圆的角：

$$
\theta = \frac{l}{r}
$$

其中$l$是弧长，以弧度制计算，总共有$2\pi$

对于球体的立体角

$$
\Omega = \frac{A}{r^2}
$$

其中$A$是球面上的面积，以弧度制计算，总共有$4\pi$

其微分如下

$$
dA=(rd\theta)(rsin\theta d\phi)=r^2sin\theta d\theta d\phi
$$

其中$\theta$是与$z$轴的夹角，$\theta$是与$x$轴的夹角。

$$
d\omega=\frac{dA}{r^2}=sin\theta d\theta d\phi
$$

所以有

$$
\Phi = \int_{S^2}Id\omega=4\pi I
$$

$$
I = \frac{\Phi}{4\pi}
$$

从实践上来看，立体角是一个单位向量

### 辐射通量密度、辐照度(Irradiance)

指辐射通量对每单位面积的量，注意传播方向，或者分量传播方向与那个单位平面垂直。或者说每投影单位面积。

$$
E(x)\equiv\frac{d\Phi(x)}{dA}
$$

对于可见光，单位为$lm/m^2=lux$，对于一般电磁波，单位为$W/m^2$

要求传播方向与平面垂直，和前面提到的Lambert's Cosine Law是相应的。

### 辐射率(Radiance)

是辐射强度对于每投影单位面积的量。

$$
L(p,\omega) = \frac{d^2\Phi(p,\omega)}{d\omega dAcso\theta}
$$

其中$\theta$是传播方向与平面法线的夹角。

对于可见光，单位是$cd/m^2=lm/(sr\cdot m^2)=nit$。对于一般电磁波，单位是$W/(sr\cdot m^2)$

显然可以推断出，辐射率是辐射通量在每立体角和每投影面积的量，也是辐照度每立体角的量。

#### 入射辐射率(Incident Radiance)

是辐射度对每单位立体角的量，其中辐射是到达平面。

$$
L(p,\omega)=\frac{dE(p)}{d\omega cos\theta}
$$

#### 出射辐射率(Exiting Radiance)

是辐射强度对每单位投影面积的量，其中辐射是从平面发出

$$
L(p,\omega) = \frac{dI(p,\omega)}{dAcos\theta}
$$

### Radiance和Irradiance

$$
dE(p,\omega) = L_i(p,\omega)cos\theta d\omega
$$

$dE(p,\omega)$，指的就是每个立体角的Irradiance，也即$dA$接收到$d\omega$发来的能量。而计算各个方向上的$dE$之和，算出来的就是Radiance，也即$dA$从各个方向接收的能量。这个各个方向通常指的是单位半球的各个立体角。单位半球记作$H^2$。

$$
E(p)=\int_{H^2}L_i(p,\omega)cos\theta d\omega
$$

### 双向反射分布函数(Bidirectional Reflectance Distribution Function)

#### 在某一点的反射过程

首先，从某个立体角$\omega_i$射过来的Radiance，变为了某个面积$dA$的能量。

然后，这个能量从这个面积再辐射出Radiance到任意其他立体角$\omega$

入射的Radiance是$dE(\omega_i)=L(\omega_i)cos\theta(i)d\omega_i$

出射的Radiance，对于某个立体角是$dL_r(\omega_r)$

而BRDF就是描述了，会有多少光从每个入射光线，反射到某个立体角$\omega_r$。

$$
f_r(\omega_i\to\omega_r)=\frac{dL_r(\omega_r)}{dE_i(\omega_i)}=\frac{dL_r(\omega_r)}{L(\omega_i)cos\theta(i)d\omega_i}
$$

单位是$1/sr$

#### 反射方程

$$
L_r(p,\omega_r)=\int_{H^2}f_r(p,\omega_i\to\omega_r)L_i(p,\omega_i)cos\theta_i d\omega_i
$$

这个方程会遇到一些问题：如果入射光也有其他物体的反射光，会导致递归方程。

#### 渲染方程

如果一个物体自己会发光，反射方程需要添加一项。

$$
L_o(p,\omega_o)=L_e(p,\omega_o)\\
+\int_{\Omega^+}L_i(p,\omega_i)f_r(p,\omega_i,\omega_o)(n\cdot w_i) d\omega_i
$$

简化的写法可以写作

$$
L(u)=e(u)+\int L(v)K(u,v)dv
$$

再进一步可以写作算子形式

L = E+KL

可以被离散化成一个简单的矩阵方程，$L,E$是向量，$K$是光传播矩阵。

如果考虑物体的反射作为新的入射光线，就会导致递归方程。将算子做如下运算

$$
L=E+KL\\
(I-K)L=E\\
L=(I-K)^{-1}E
$$

将中间的$(I-K)^{-1}$展开成幂级数

$$
L=(I+K+K^2+K^3+\cdots)E\\
L=E+KE+K^2E+K^3E+\cdots
$$

其中第一项代表光源直接射过来的光，第二项表示光源经过一次反射过来的光，第三项表示光源经过一次反射过来的光。以此类推。这也是全局光照的基础。

## 路径追踪

Whitted-Style Ray Tracing的错误之处：无法正确处理略有磨砂感的金属表面，无法正确处理漫反射。

但是渲染方程是正确的。但缺陷是，要在半球面上解定积分，以及有递归运算。

### 一个简单情况下的蒙特卡罗积分

假设我们要渲染一个像素(点)，只考虑直接照射的光线。假设这个点不发光。渲染方程如下：

$$
L_o(p,\omega_o)=\int_{\Omega^+}L_i(p,\omega_i)f_r(p,\omega_i,\omega_o)(n\cdot \omega_i) d\omega_i
$$

蒙特卡洛积分中的$f(x)$为：

$$
L_i(p,\omega_i)f_r(p,\omega_i,\omega_o)(n\cdot \omega_i) 
$$

蒙特卡洛积分中的pdf是（假设均匀采样）：

$$
p(\omega_i)=\frac{1}{2\pi}
$$

所以最终：

$$
L_o(p,\omega_o)=\int_{\Omega^+}L_i(p,\omega_i)f_r(p,\omega_i,\omega_o)(n\cdot \omega_i) d\omega_i
$$

$$
\approx\frac{1}{N}\sum_{i=1}^N\frac{L_i(p,\omega_i)f_r(p,\omega_i,\omega_o)(n\cdot \omega_i)}{p(\omega_i)}
$$

这对于直接光照来说是一个正确的光栅化算法。

伪代码如下

```
shade(p,wo)
    Randomly choose N directions wi~pdf
    Lo=0.0
    For each wi
        Trace a ray r(p,wi)
        If ray r hit the light
            Lo+=(1/N)*L_i*f_r*cosine/pdf(wi)
    Return Lo
```

### 全局光照情况下

#### 解决反射导致的光线数量爆炸

间接光照也要考虑的情况下，我们很容易得到一个伪代码：

```
shade(p,wo)
    Randomly choose N directions wi~pdf
    Lo=0.0
    For each wi
        Trace a ray r(p,wi)
        If ray r hit the light
            Lo+=(1/N)*L_i*f_r*cosine/pdf(wi)
        Else If ray r hit an object at q
            Lo+=(1/N)*shade(q,-wi)*f_r*cosine/pdf(wi)
    Return Lo
```

这会导致一个问题，如果是一个不太光滑的物体，第一次反射后有100个方向的漫反射，一直递归下去，就有$100^n$条光线，这是不能接受的。而且，显然只有在，反射后只有1个方向的光线时，才不会出现指数爆炸。此时叫做路径追踪。伪代码如下：

```
shade(p,wo)
    Randomly choose ONE directions wi~pdf
    Trace a ray r(p,wi)
    If ray r hit the light
        Return L_i*f_r*cosine/pdf(wi)
    Else if ray r hit an object at q
        Return shade(q,-wi)*f_r*cosine*pdf(wi)
    
```

如果反射后的光线方向不止一个，那么称作分布式光线追踪，这已经是一个过时的想法了。

虽然解决了指数爆炸问题，但是，这会导致很多的噪声。

解决噪声的办法是，对于每一个像素，发出更多的光线，最后取Radiance的平均。伪代码如下

```
ray_generation(camPos, pixel)
    Uniformly choose N sample position within the pixel
    pixel_radiance = 0.0
    For each sample in the pixel
        Shoot a ray r(camPos, cam_to_sample)
        If ray r hit the scene at p
            pixel_radiance+=1/N*shade(p,sample_to_cam)
    Return pixel_radiance
```

#### 解决光线反射次数无限的问题

多次反射导致指数爆炸的问题解决了，还有一个问题是递归问题。

shade函数并没有给出明确的函数停止的条件，这会导致无限递归。且粗暴的设定一个硬性条件则会导致能量的丢失。

有一个俄罗斯转盘（RR）算法。

之前，我们总是对一个像素射出光线来得到着色结果$Lo$。

现在，假设我们手动设定了一个概率$P(0<P<1)$。

当我们遇到概率为$P$的情况时，射出一个光线，并且返回着色结果，再讲这个结果除以$P$，得到$Lo/P$

遇到概率为$1-P$的情况时，不射出光线，得到的结果为$0$。

最后的能量为：

$$
E=P*(Lo/P)+(1-P)*0=Lo
$$

伪代码如下

```
shade(p,wo)
    Manually specify a probability P_RR
    Randomly select ksi in a uniform dist. in[0,1]
    If (ksi>P_RR) Return 0.0

    Randomly choose ONE directions wi~pdf
    Trace a ray r(p,wi)
    If ray r hit the light
        Return L_i*f_r*cosine/pdf(wi)/P_RR
    Else if ray r hit an object at q
        Return shade(q,-wi)*f_r*cosine*pdf(wi)/P_RR    
```

#### 提升效率

在着色点上反射光线，如果光源面积很大，则几根光线就可以打到光源。如果光源很小、接近点光源，则要很多光线才能打到。因为我们是均匀地往四周反射光线，或者说，我们的pdf是一个常数。

可以考虑将在半球上的积分转化为在光源上的积分。

需要找到$d\omega$（半球立体角）和$dA$（光源上的面积）的关系。

显然，这是一个投影关系，又因为半球的半径是$1$，有如下关系

$$
d\omega = \frac{dAcos\theta'}{||x'-x||^2}
$$

其中$x'$是光源微平面的坐标，$x$是着色点的坐标，$\theta'$是光源微平面法向量和$x-x'$向量的夹角。

渲染方程重写为

$$
L_o(x,\omega_o)=
\int_{\Omega^+}L_i(x,\omega_i)f_r(x,\omega_i,\omega_o)cos\theta d\omega_i\\
$$

$$
=\int_A L_i(x,\omega_i)f_r(x,\omega_i,\omega_o)\frac{cos\theta cos\theta'}{||x'-x||^2} dA
$$

$\theta$是着色点微平面的法向量和$x'-x$向量的夹角。

蒙特卡洛积分的$f(x)$变为

$$
L_i(x,\omega_i)f_r(x,\omega_i,\omega_o)\frac{cos\theta cos\theta'}{||x'-x||^2}
$$

pdf变为$1/A$

之后，我们对光源不使用RR，对其他反射使用RR。

另外，转化为$dA$后，要考虑中间是否有物体挡住光源。

最终伪代码如下

```
shade(p,wo)
    # Contribution from the light source.
    L_dir = 0.0
    Uniformly sample the light at x' (pdf_light = 1/A)
    Shoot a ray from p to x'
    If the ray is not blocked in the middle
        L_dir = L_i*f_r*cos1*cos2/|x'-p|^2/pdf_light

    # Contribution from other reflectors
    L_indir = 0.0
    Test Russian Roulette with probability P_RR
    Uniformly samplethe hemisphere toward wi(pdf_hemi = 1/2pi)
    Trace a ray r(p,wi)
    If ray r hit a non-emitting object at q
      L_indir = shade(q,-wi)*f_r*cos1/pdf_hemi/P_RR
    
    Return L_dir+L_indir

```


