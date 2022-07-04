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

## 离散概率

TODO

## 连续概率

TODO

## 蒙特卡罗积分

TODO

