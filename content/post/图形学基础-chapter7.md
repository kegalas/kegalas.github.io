---
title: "计算机图形学基础（第五版）-第七章笔记"
date: 2022-06-27T16:25:19+08:00
draft: false
tags: [图形学]
categories: 图形学
mathjax: true
markup: pandoc
---

## 放缩

$$
scale(s_x,s_y)=
\begin{bmatrix}
s_x  & 0\\
0  & s_y
\end{bmatrix}
$$

$$
\begin{bmatrix}
s_x  & 0\\
0  & s_y
\end{bmatrix}
\begin{bmatrix}
x\\
y
\end{bmatrix}=
\begin{bmatrix}
s_xx\\
s_yy
\end{bmatrix}
$$

例如长宽各缩小为0.5倍，有

$$
scale(0.5,0.5)=
\begin{bmatrix}
0.5  & 0\\
0  & 0.5
\end{bmatrix}
$$

## 切变

切变，想象一个由四根木条和四个钉子组装而成的正方形，我们可以将其“推”成一个平行四边形。但是在图形学中，长和高不变，意味着有两条边的长度会发生变化。

$$
shear-x(s)=\begin{bmatrix}
1  & s\\
0  & 1
\end{bmatrix},
shear-y(s)=\begin{bmatrix}
1  & 0\\
s  & 1
\end{bmatrix}
$$

另外一种关于切变的理解是，仅仅对着x坐标或者y坐标进行了旋转操作，例如正向旋转（逆时针）$\phi$，则有

$$
\begin{bmatrix}
1  & tan\phi\\
0  & 1
\end{bmatrix}
or
\begin{bmatrix}
1  & 0\\
tan\phi  & 1
\end{bmatrix}
$$

## 旋转

顺时针转过$\phi$，则

$$
rotate(\phi) = 
\begin{bmatrix}
cos\phi  & -sin\phi\\
sin\phi  & cos\phi
\end{bmatrix}
$$

## 镜像

即关于$x$轴或$y$轴将整个图像颠倒过来，有

$$
reflect-y = 
\begin{bmatrix}
-1  & 0\\
0  & 1
\end{bmatrix},
reflect-x = 
\begin{bmatrix}
1  & 0\\
0  & -1
\end{bmatrix}
$$

## 线性变换的复合

对于两个变换$\bm S,R$

$$
first,\bm v_2=\bm{Sv}_1,then,\bm v_3=\bm{Sv}_2
$$

那么就可以写作

$$
\bm v_3=\bm R(\bm{Sv}_1)
$$

根据结合律，写作

$$
\bm v_3=(\bm{RS})\bm{v}_1
$$

则$\bm{M=RS}$就是复合变换，其中变换顺序是从右到左的。

## 线性变换的拆分

TODO

## 三维线性变换

总体来说，三维线性变换就是二维的扩展

$$
scale(s_x,s_y,s_z)=
\begin{bmatrix}
s_x  & 0 & 0\\
0  & s_y & 0\\
0  & 0 & s_z
\end{bmatrix}
$$

$$
rotate-z(\phi)=
\begin{bmatrix}
cos\phi  & -sin\phi & 0\\
sin\phi  & cos\phi & 0\\
0  & 0 & 1
\end{bmatrix}
$$

$$
rotate-x(\phi)=
\begin{bmatrix}
1  & 0 & 0\\
0  & cos\phi & -sin\phi\\
0  & sin\phi & cos\phi
\end{bmatrix}
$$

$$
rotate-y(\phi)=
\begin{bmatrix}
cos\phi  & 0 & sin\phi\\
0  & 1 & 0\\
-sin\phi  & 0 & cos\phi
\end{bmatrix}
$$

$$
shear-x(d_y,d_z)=
\begin{bmatrix}
1 & d_y & d_z\\
0 & 1 & 0\\
0 & 0 & 1
\end{bmatrix}
$$

未完待续

## 平移

平移不能被写作矩阵的形式，所以它不是线性变换。

$$
\begin{bmatrix}
x'\\
y'
\end{bmatrix}=
\begin{bmatrix}
a  & b\\
c  & d
\end{bmatrix}
\begin{bmatrix}
x\\
y
\end{bmatrix}+
\begin{bmatrix}
t_x\\
t_y
\end{bmatrix}
$$

但是我们仍然有办法用矩阵来表示平移，这需要转化为齐次矩阵。

为坐标（二维坐标）添加第三维，

1. 对于二维点，$(x,y,1)^T$
2. 对于二维向量，$(x,y,0)^T$

于是我们有平移变换如下

$$
\begin{bmatrix}
x'\\
y'\\
w'
\end{bmatrix}=
\begin{bmatrix}
1  & 0 & t_x\\
0  & 1 & t_y\\
0 & 0 & 1
\end{bmatrix}\cdot
\begin{bmatrix}
x\\
y\\
1
\end{bmatrix}=
\begin{bmatrix}
x+t_x\\
y+t_y\\
1
\end{bmatrix}
$$

并且这样定义点和向量有好处，向量加向量是向量，点减点是向量，点加向量是向量。

平移一个向量，对于自由向量来说，起点都是远点，所以平移向量不会改变向量的坐标表示。为此设置第三维为0是合理的。

同样地，在复合变换时，用矩阵乘法来复合。

## 三维平移

$$
\bm T(t_x,t_y,t_z)=\
\begin{bmatrix}
1 & 0 & 0 & t_x\\
0 & 1 & 0 & t_y\\
0 & 0 & 1 & t_z\\
0 & 0 & 0 & 1
\end{bmatrix}
$$

## 仿射变换

即线性变换加上一个平移

$$
\begin{bmatrix}
x'\\
y'
\end{bmatrix}=
\begin{bmatrix}
a & b\\
c & d
\end{bmatrix}\cdot
\begin{bmatrix}
x\\
y
\end{bmatrix}+
\begin{bmatrix}
t_x\\
t_y
\end{bmatrix}
$$

$$
\begin{bmatrix}
x'\\
y'\\
1
\end{bmatrix}=
\begin{bmatrix}
a  & b & t_x\\
c  & d & t_y\\
0 & 0 & 1
\end{bmatrix}\cdot
\begin{bmatrix}
x\\
y\\
1
\end{bmatrix}
$$

显然知道，这个矩阵先进行线性变换，再进行平移。

## 逆变换

变换$\bm M^{-1}$是变换$\bm M$在矩阵和几何意义上的逆变换。

TODO

## 坐标变换

TODO
