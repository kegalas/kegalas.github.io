---
title: 从零开始的软渲染器 坐标变换与视角
date: 2023-08-04T22:53:54+08:00
draft: true
tags:
  - 图形学
  - 渲染
categories: 图形学
mathjax: true
markup: goldmark
image: cover.jpg
---

我们的模型并不会停留在原地不动，比如当我们在游戏中前后左右移动时，其他物体相对于我们是在平移的。又比如在游戏中移动鼠标看向不同的地方，其他物体和我们距离保持不变，但方向却改编了。

为此，研究物体的坐标变换是十分有必要的。

我们之前都只是简单的把物体的$z$轴坐标去掉，只用$x,y$轴坐标来画图。这导致的问题是，我们无法实现近大远小的效果，美术上也把这个称为透视原理。为此，我们还必须研究一种方法，让渲染考虑$z$轴坐标，实现对于透视的模拟。

# 坐标变换

一般来说，我们主要用到平移、旋转、缩放，其他的如切变、镜像等并不是很常用到，暂时不做介绍。另外，我们最开始打算的就是在三维空间中运行，所以就不介绍更简单的二维情况了

## 缩放

从最简单的缩放讲起，如果一个点是$(x,y,z)$，那么把他缩放$k$倍，就是简单的$(kx,ky,kz)$。这实在是过于显然的结论。用矩阵实现的话，这个变换的矩阵是

$$
\begin{bmatrix}
 k & 0 & 0\\
 0 & k & 0\\
 0 & 0 & k
\end{bmatrix}
$$

当然，如果你想让各个轴上缩放比例不一样，例如$x$轴缩放$k_x$倍，$y$轴缩放$k_y$倍，$z$轴缩放$k_z$倍，那么

$$
\begin{bmatrix}
 k_x & 0 & 0\\
 0 & k_y & 0\\
 0 & 0 & k_z
\end{bmatrix}
$$

其作用在某一点$(x,y,z)^T$上，结果为

$$
\begin{bmatrix}
 k_x & 0 & 0\\
 0 & k_y & 0\\
 0 & 0 & k_z
\end{bmatrix}
\begin{bmatrix}
 x\\
 y\\
 z
\end{bmatrix}=
\begin{bmatrix}
 k_xx\\
 k_yy\\
 k_zz
\end{bmatrix}
$$

写成四维的形式就是

$$
\begin{bmatrix}
 k_x & 0 & 0 & 0\\
 0 & k_y & 0 & 0\\
 0 & 0 & k_z & 0\\
 0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
 x\\
 y\\
 z\\
 1
\end{bmatrix}=
\begin{bmatrix}
 k_xx\\
 k_yy\\
 k_zz\\
 1
\end{bmatrix}
$$

## 平移

事实上，三维矩阵是无法进行平移的，我们必须加入一维。这也是之前提到用四维向量表示三维向量的原因所在（TODO，检查我真的讲过这个吗）。

具体而言，我们引入下面这个矩阵

$$
\begin{bmatrix}
 1 & 0 & 0 & t_x\\
 0 & 1 & 0 & t_y\\
 0 & 0 & 1 & t_z\\
 0 & 0 & 0 & 1
\end{bmatrix}
$$

对于任意一点$(x,y,z,1)^T$应用这个变换，得到

$$
\begin{bmatrix}
 1 & 0 & 0 & t_x\\
 0 & 1 & 0 & t_y\\
 0 & 0 & 1 & t_z\\
 0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
 x\\
 y\\
 z\\
 1
\end{bmatrix}=
\begin{bmatrix}
 x+t_x\\
 y+t_y\\
 z+t_z\\
 1
\end{bmatrix}
$$

我们很自然地发现，这就是真正的平移，每个坐标都位移了$t$的距离。

## 旋转

## 复合变换
