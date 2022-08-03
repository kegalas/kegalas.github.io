---
title: "计算机图形学基础（第五版）-第二章笔记"
date: 2022-06-27T15:17:29+08:00
draft: false
tags: [图形学]
categories: 图形学
mathjax: true
markup: pandoc
---

## Cartesian Product（笛卡尔积）

内容同离散数学。

## 几个特别的集合

$S^2$，在单位球面上的三维点的集合。

## 映射、函数、反函数

内容同离散数学。

## 三角学

书中提到的正弦余弦定理、半角公式、和差公式都已在高中学过，记录一些没学过的。

**正切定理**

$$
\frac{a+b}{a-b}=\frac{tan\left(\frac{A+B}{2}\right)}{tan\left(\frac{A-B}{2}\right)}
$$

**海伦公式**

$$
S = \frac{1}{4}\sqrt{(a+b+c)(-a+b+c)(a-b+c)(a+b-c)}
$$

### 重心坐标系

可以用三个点$\bm{a,b,c}$表示三角形。

假设三点不共线，则$\bm{c-a},\bm{b-a}$线性无关，即可以作为二维坐标的一组基底。

二维空间中任何一点都可以由

$$
\bm{p}=\bm a+\beta(\bm{b-a})+\gamma(\bm{c-a})
$$

表示

整理得

$$
\bm{p}=(1-\beta-\gamma)\bm a+\beta\bm{b}+\gamma\bm{c}
$$

令

$$
\alpha\equiv 1-\beta-\gamma
$$

则有

$$
\bm p(\alpha,\beta,\gamma)=\alpha\bm a+\beta\bm{b}+\gamma\bm{c}
$$

其中

$$
\alpha+\beta+\gamma = 1
$$

一个点在三角形内部当且仅当

$$
0<\alpha<1,0<\beta<1,0<\gamma<1
$$

同时成立。也可以用向量叉乘的办法判断是否在内部。

对于三角形三个点$A,B,C$，平面中一点$(x,y)$可以表示为

$$
(x,y) = \alpha A+\beta B+\gamma C
$$

$$
\alpha+\beta+\gamma=1
$$

其中有

$$
\alpha = \frac{-(x-x_B)(y_C-y_B)+(y-y_B)(x_C-x_B)}{-(x_A-x_B)(y_C-y_B)+(y_A-y_B)(x_C-x_B)}
$$

$$
\beta = \frac{-(x-x_C)(y_A-y_C)+(y-y_C)(x_A-x_C)}{-(x_B-x_C)(y_A-y_C)+(y_B-y_C)(x_A-x_C)}
$$

$$
\gamma = 1-\alpha-\beta
$$

运用重心坐标系进行颜色插值见第九章笔记。

## 向量

同高中和线性代数

## 积分

内容同高数。不过在计算机图形学里我们更注重数值而不是分析。

## 曲线、曲面

内容同高数。

## Linear Interpolation（线性内插） 

$$
f(x)=y_i+\frac{x-x_i}{x_{i+1}-x_i}(y_{i+1}-y_i)
$$

## 概率论

### 随机变量

$X$，表示任意一个可能取值。

### 概率密度函数(PDF)

$X\sim p(x)$，表示随机过程取值为$x$的相对概率。

### 概率的一些属性

1. $p_i\geq 0$
2. $\sum_{i=1}^np_i=1$

### 期望

$$
E[X] = \sum_{i=1}^nx_ip_i
$$

### 连续的情况

当随机变量$X$可以取一个连续的区间上的值。

显然此时也会有连续的概率密度函数。

并且

$$
p(x)\geq 0,\int p(x)dx=1
$$

$$
E[x]=\int xp(x)dx
$$

### 随机变量的函数

随机变量的函数也是一个随机变量。

$$
X\sim p(x)\\
Y=f(X)
$$

$$
E[Y]=E[f(X)]=\int f(x)p(x)dx
$$

## 蒙特卡罗积分

蒙特卡洛积分的目的：想计算一个定积分，但是难以从分析意义上解出，希望在数值上求解。

方法：通过平均函数值的随机样本来估计函数的积分。

定义定积分如下：

$$
\int_a^b f(x)dx
$$

随机变量如下

$$
X_i\sim p(x)
$$

则蒙特卡洛估计值是：

$$
F_N=\frac{1}{N}\sum_{i=1}^N\frac{f(X_i)}{p(X_i)}
$$

如果随机变量是均匀的，或者说

$$
X_i\sim p(x)=C
$$

$C$是一个常数

那么，

$$
\int_a^b p(x)dx=1
$$

$$
\int_a^b Cdx=1
$$

$$
C=\frac{1}{b-a}
$$

此时基础蒙特卡洛估计值(Basic Monte Carlo Estimator)为

$$
F_N=\frac{b-a}{N}\sum_{i=1}^N f(X_i)
$$


