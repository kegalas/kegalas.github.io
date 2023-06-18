---
title: "最优化理论学习笔记"
date: 2023-06-16T19:47:47+08:00
draft: false
tags: [大学, 数学, 最优化]
description: 最优化理论、算法的学习笔记
categories: 数学
mathjax: true
markup: pandoc
image: "cover.jpg"
---

# 读前警示

出于Obsidian不支持\bm的标签，也出于我打字方便、教科书也没用加粗的原因，从某一段开始的瞬间，很多地方都不会使用粗体来表示向量。这其实无所谓，我们可以把标量看做一维向量。我们的大部分讨论都是在向量之上的，除了部分参数是标量。大部分矩阵是大写字母。

阅读时应结合前后文理解一个符号是什么意思。

# 数学基础概念、定义

## 向量内积

在$n$维线性空间$R^n$中，$\bm a = [a_1,a_2,\cdots,a_n]^T\in R^n,\bm b = [b_1,b_2,\cdots,b_n]^T\in R^n$，则

$$
\bm a^T b = a_1b_1+a_2b_2+\cdots,a_nb_n
$$

称作向量$\bm a,\bm b$的内积。

内积满足交换律，线性，以及正定性：$\bm a^T \bm a\geq 0$，当且仅当$\bm a=\bm 0$时，取等号。

其他各种性质都在线性代数中学过了，这里只是简单回顾。

## 向量范数

称一个从向量空间$R^n$到实数域$R$的非负函数$||\cdot||$为范数，如果它满足：

1. 正定性：对于所有的$\bm v\in R^n$，有$||\bm v||\geq 0$，且$||\bm v||=0$当且仅当$\bm v=0$
2. 齐次性：对于所有的$\bm v\in R^n$和$\alpha\in R$，有$||\alpha\bm v||=|\alpha|||\bm v||$
3. 三角不等式：对于所有的$\bm v,\bm w\in R^n$，有$||\bm v+\bm w||\leq ||\bm v||+||\bm w||$

对于$\bm v = (v_1,v_2,\cdots,v_n)^T$，常见的向量范数为$\mathscr{l}_n$范数$(p>=1)$

$$
||v||_p = (|v_1|^p+|v_2|^p+\cdots+|v_3|^p)^{1/p}
$$

当$p=\infty$时，$\mathscr{l}_\infty$定义为

$$
||v||_\infty = \max_i|v_i|
$$

一些地方也可以见到$0$-范数，其表示分量中不为零的个数。

最常见的是$2$-范数，又称欧几里得范数，常常$||\cdot||$省略角标不写表示$2$-范数。

## 二次型

$n$维二次函数可以表示如下

$$
f(x_1,x_2,\cdots,x_n)=a_{11}x_1^2+a_{12}x_1x_2+\cdots+a_{1n}x_1x_n+a_{21}x_2x_1+\cdots+a_{nn}x^2_n
$$

其可以表示为向量矩阵形式

$$
\sum^n_{i=1}\sum^n_{j=1}a_{ij}x_ix_j=\bm x^T A \bm x
$$

这个$A$矩阵是个实对称矩阵，其元素$a_{ij}=a_{ji}$，是对应的$x_ix_j$项系数的一半（原式子出于简单起见没有把同类项合并）。

## 正定矩阵

如果正定二次型$f(\bm x)=\bm x^TA\bm x$，对于一组不全为零的数$x_1,x_2,\cdots,x_n$，恒有$f(x_1,x_2,\cdots,x_n)>0$，则称$f$正定，$A$为正定矩阵。若换成$\geq$，则称为半正定矩阵，若换成$\leq$，则称为半负定矩阵。

有两个判定定理

1. $A$正定的充要条件为$A$的特征值都大于$0$
2. $A$正定的充要条件为$A$的所有顺序主子式都大于$0$

## 方向导数

设$f:R^n\to R$在点$\bm x$处可微，$\bm p$是固定不变的非零向量，$\bm e$是方向$\bm p$上的单位向量，则称极限

$$
\dfrac{\partial f(\bm x)}{\partial \bm p} = \lim_{t\to 0^+}\dfrac{f(\bm x+t\bm e)-f(\bm x)}{t}
$$

为函数$f(x)$在点$\bm x$处沿$\bm p$方向的方向导数，记为$\dfrac{\partial f(\bm x)}{\partial\bm p}$。当其大于零时，函数在该点沿此方向上升，小于零时沿此方向下降。

## 梯度

给定函数设$f:R^n\to R$，且$f$在点$\bm x$的一个领域内有意义，若存在向量$\bm g\in R^n$满足

$$
\lim_{\bm p\to \bm 0}\dfrac{f(\bm x+\bm p)-f(\bm x)-\bm g^T\bm p}{||\bm p||}=0
$$

其中这个范数可以是任意向量范数，就称$f$在点$\bm x$可微，此时$\bm g$是$f$在点$\bm x$处的梯度，记作$\nabla f(x)$。可以计算，若令$\bm p=\epsilon \bm e_i$，$\bm e_i$是第$i$个分量为$1$的单位向量，可知$\nabla f(x)$的第$i$个分量即为$\dfrac{\partial f(x)}{\partial x_i}$。因此

$$
\nabla f(x) = \bigg[\dfrac{\partial f(x)}{\partial x_1},\dfrac{\partial f(x)}{\partial x_2},\cdots,\dfrac{\partial f(x)}{\partial x_n}\bigg]^T
$$

一个非常浅显的事实是，一维向量函数的梯度就是它的一阶导数。

梯度方向是函数在该点上升的最快方向，反方向是最速下降方向。

梯度与方向导数的关系为

$$
\dfrac{\partial f(\bm x)}{\partial \bm p}=\nabla f(\bm x)^T\bm e
$$

其中$\bm e$是$\bm p$方向上的单位向量。

$\bigg|\dfrac{\partial f(\bm x)}{\partial \bm p}\bigg|\leq |\nabla f(\bm x)^T|$

梯度有几条显然的性质：

1. 若$\nabla f(\bm x)^T\bm p<0$，则$\bm p$的方向是函数$f$在$\bm x$处的下降方向
2. 若$\nabla f(\bm x)^T\bm p>0$，则$\bm p$的方向是函数$f$在$\bm x$处的上升方向
3. 梯度正交的方向变化率为零

## 海瑟矩阵

用一个简单的比方，梯度和海瑟矩阵的关系，就相当于一阶导数和二阶导数的关系。

给定函数$f:R^n\to R$，若其在点$\bm x$处的二阶偏导数$\dfrac{\partial^2 f(\bm x)}{\partial \bm x_i\partial \bm x_j},i,j=1,2,\cdots,n$都存在，则

$$
\nabla^2f(\bm x) = 
\begin{bmatrix}
 \dfrac{\partial^2 f(\bm x)}{\partial x_1^2} & \dfrac{\partial^2 f(\bm x)}{\partial x_1\partial x_2} & \dfrac{\partial^2 f(\bm x)}{\partial x_1\partial x_3} & \cdots & \dfrac{\partial^2 f(\bm x)}{\partial x_1\partial x_n}\\
 \dfrac{\partial^2 f(\bm x)}{\partial x_2\partial x_1} & \dfrac{\partial^2 f(\bm x)}{\partial x_2^2} & \dfrac{\partial^2 f(\bm x)}{\partial x_2\partial x_3} & \cdots & \dfrac{\partial^2 f(\bm x)}{\partial x_2\partial x_n}\\
 \vdots & \vdots & \vdots &  & \vdots\\
 \dfrac{\partial^2 f(\bm x)}{\partial x_n\partial x_1} & \dfrac{\partial^2 f(\bm x)}{\partial x_n\partial x_2} & \dfrac{\partial^2 f(\bm x)}{\partial x_n\partial x_3} &  & \dfrac{\partial^2 f(\bm x)}{\partial x_n^2}
\end{bmatrix}
$$

称为$f$在点$x$处的海瑟矩阵。

## 泰勒展开

给定函数$f:R^n\to R$，具有二阶连续偏导数，则

$$
f(\bm x+\bm p) = f(\bm x) + \nabla f(\bm x)^T\bm p + \dfrac{1}{2}\bm p^T\nabla^2 f(\bm x)^T\bm p + o(||\bm p||^2)
$$

其中$o(||p||^2)$当$||p||^2\to 0$时，是关于$||p||^2$的高阶无穷小量。

## 邻域

对于任意给定的实数$\delta>0$，满足不等式$||\bm x-\bm x_0||<\delta$的$\bm x$的集合称为点$\bm x_0$的邻域，记为

$$
N(\bm x_0,\delta) = \{\bm x|\ ||\bm x-\bm x_0||<\delta,\delta>0\}
$$

## 极小点

设$f:D\subseteq R^n\to R$，若存在点$\bm x^*\in D$和数$\delta>0$，$\forall \bm x\in N(\bm x^*,\delta)\cap D$，都有$f(\bm x^*)\leq f(\bm x)$，则称$\bm x^*$为$f(\bm x)$的（非严格）局部极小点。

设$f:D\subseteq R^n\to R$，若存在点$\bm x^*\in D$和数$\delta>0$，$\forall \bm x\in N(\bm x^*,\delta)\cap D$，但$\bm x\neq \bm x^*$，都有$f(\bm x^*)< f(\bm x)$，则称$\bm x^*$为$f(\bm x)$的严格局部极小点。

设$f:D\subseteq R^n\to R$，若存在点$\bm x^*\in D$和数$\delta>0$，$\forall \bm x\in D$，都有$f(\bm x^*)\leq f(\bm x)$，则称$\bm x^*$为$f(\bm x)$的（非严格）全局极小点。

设$f:D\subseteq R^n\to R$，若存在点$\bm x^*\in D$和数$\delta>0$，$\forall \bm x\in D$，但$\bm x\neq \bm x^*$，都有$f(\bm x^*)\leq f(\bm x)$，则称$\bm x^*$为$f(\bm x)$的严格全局极小点。

如果$\bm x^*$是极小点并且是$D$的内点，则$\nabla f(\bm x^*)=0$。这是一个必要非充分条件。

但如果$f$有连续二阶偏导数，$\bm x^*$是一个驻点，并且$\nabla^2 f(\bm x^*)$是正定的，则$\bm x^*$是$f(\bm x)$的严格局部极小点。

## 驻点

设$f:D\subseteq R^n\to R$，$\bm x^*$是$D$的内点，若$\nabla f(\bm x^*)=0$，则称$\bm x^*$为$f(\bm x)$的驻点。

## 锥

设集合$C\subset R^n$，若对$\forall\bm x\in C$以及$\forall \lambda\geq 0$均有$\lambda\bm x\in C$，则称$C$为锥

## 凸组合

设$\bm x_1,\bm x_2,\cdots,\bm x_l$是$R^n$中的$l$个已知点。若对于某点$\bm x\in R^n$，存在常数$\lambda_1,\lambda_2,\cdots,\lambda_l\geq 0$，且$\sum\lambda_i=1$，使得$\bm x=\sum\lambda_i\bm x_i$，则称$\bm x$是$\bm x_1,\bm x_2,\cdots,\bm x_l$的凸组合。如果$\lambda_1,\lambda_2,\cdots,\lambda_l>0$且$\sum\lambda_i=1$，则称为严格凸组合。

## 凸集

对于$C\subset R^n$中的任意两个点$\bm x_1, \bm x_2$，如果连接两个点的线段在$C$内，则称$C$为凸集。即

$$
\bm x_1,\bm x_2\in C\Rightarrow\theta \bm x_1+(1-\theta)\bm x_2\in C,\forall 0\leq\theta\leq 1
$$
**定理1**

任意一组凸集的交仍然是凸集。

特别的，规定空集是凸集。容易验证，空间$R^n$、半空间、超平面、直线、点、球都是凸集。

**凸集分离定理**

设$C_1,C_2$是$R^n$中两个非空的集合，$H=\{\bm x|\bm p^T\bm x=\alpha\}$为超平面，如果对$\forall\bm x\in C_1$都有$\bm p^T\bm x\geq \alpha$，对$\forall x\in C_2$都有$\bm p^T\bm x\leq \alpha$（或二者正好相反），则称超平面$H$分离集合$C_1,C_2$

## 半空间

设$\bm a\in R^n,\bm a\neq\bm 0,b\in R$，则集合

$$
\{\bm x|\bm a^T\bm x>b,\bm x\in R^n\}
$$

称为$R^n$的半空间。

有限个半空间的交$\{\bm x|\bm A\bm x\leq \bm b\}$称为多面集，其中$\bm A$为$m\times n$矩阵，$\bm b$为$m$维向量。

## 凸函数

**广义实值函数**

令$\bar R\xlongequal{\text{def}}R\cup\{\pm\infty\}$为广义实数空间，则映射$f:R^n\to\bar R$为广义实值函数。

规定$-\infty<a<+\infty,\forall a\in R$，以及$(+\infty)+(+\infty)=+\infty,(+\infty)+a=+\infty,\forall a\in R$

**适当函数**

给定广义实值函数$f$和非空集合$\mathcal{X}$，如果存在$x\in\mathcal{X}$使得$f(x)<+\infty$，并且对任意的$x\in\mathcal{X}$，都有$f(x)>-\infty$，那么称函数$f$关于集合$\mathcal{X}$是适当的。

**凸函数**

设函数$f$为适当函数，如果$\text{dom} f$是凸集，且

$$
f(\theta x+(1-\theta)y)\leq \theta f(x)+(1-\theta)f(y)
$$

对所有$x,y\in\text{dom}f,0\leq\theta\leq 1$都成立，则称$f$是凸函数。

如果$f(\theta x+(1-\theta)y)< \theta f(x)+(1-\theta)f(y)$，则称为严格凸函数。

**性质1**

设$f_1,f_2$是凸集$C$上的凸函数，则函数$f_1+f_2$在$S$上也是凸函数。

**性质2**

设$f$是凸集$C$上的凸函数，则对任意的$a\geq 0$，函数$af$也是凸函数。

## 凸函数判定

$f$是凸函数，当且仅当对任意的$x\in\text{dom}f,v\in R^n,g:R\to R$

$$
g(t)=f(x+tv),\text{dom} g = \{t|x+tv\in\text{dom} f\}
$$

是凸函数。

**一阶条件**

对于定义在凸集上的可微函数$f$，$f$是凸函数当且仅当

$$
f(y)\geq f(x)+\nabla f(x)^T(y-x),\forall x,y\in\text{dom}f
$$

严格凸函数则要求$>$号。物理意义是：任意点处的切线增量不超过函数的增量。

**梯度单调性**

设$f$为可微函数，则$f$为凸函数当且仅当$\text{dom}f$为凸集且$\nabla f$为单调映射，即

$$
(\nabla f(x)-\nabla f(y))^T(x-y)\geq 0,\forall x,y\in\text{dom}f
$$

**二阶条件**

设$f$为定义在凸集上的二阶连续可微函数，则$f$是凸函数当且仅当

$$
\nabla^2 f(x)\succeq 0,\forall x\in\text{dom}f
$$

如果$\succ$，则是严格凸函数。

或者，凸函数的充要条件是$\nabla^2 f(x)$在$\text{dom}f$上的任意点均半正定。

# 线性规划

## 图解法

1. 确定可行域
2. 确定目标函数的等值线及优化方向
3. 平行移动目标函数等值线，通过观察得到线性规划的最优解

高中已经学过很多次这个东西了，不再详细介绍。

注意到，图解法只适用于有两个变量的情况，更高维的情况下，要用单纯形法。

## 一般形式

一般形式如下

$$
\min f(x_1+x_2+\cdots+x_n) = c_1x_1+c_2x_2+\cdots+c_nx_n
$$

$$
s.t.
\left\{\begin{matrix}
a_{11}x_1+a_{12}x_2+\cdots+a_{1n}x_n = b_1\\
a_{21}x_1+a_{22}x_2+\cdots+a_{2n}x_n = b_2 \\
\vdots \\
a_{m1}x_1+a_{m2}x_2+\cdots+a_{mn}x_n = b_m\\
x_1,x_2,\cdots,x_n\geq 0
\end{matrix}\right.
$$

一共包含$n$个决策变量，$m$个约束条件。其中对目标函数既可以求最大也可以求最小，约束条件可以是$=,\leq,\geq$。决策变量$x_i$通常非负，也有其他情况。$c_j$称为价值系数，$b_j$称为资源系数，$a_{ij}$称为约束系数、技术系数。

利用矩阵，可以写作

$$
\min f(x) = Cx
$$

$$
s.t.
\left\{\begin{matrix}
Ax=b\\
x\geq 0
\end{matrix}\right.
$$

## 标准形式

标准形式有四个特征

1. 目标函数求$\min$
2. 约束条件两端用$=$连接
3. 所有$b_i$非负
4. 所有$x_i$非负

接下来介绍如何转换其他形式到标准形式（注意按顺序来）

**目标函数求最小值**

例如$\min z=x_1-2x_2+3x_3$，等价于$\max z'=-x_1+2x_2-3x_3$，其中$z'=-z$

**约束函数求最小值**

1. 例如$-2x_1+6x_2-x_3\leq3$，将其转化为

$$
-2x_1+6x_2-x_3+x_s=3
$$

$x_s$表示决策中尚未使用的那部分资源，所以称之为松弛变量。$x_s\geq 0$

2. 例如$-2x_1+6x_2-x_3\ge3$，将其转化为

$$
-2x_1+6x_2-x_3-x_s=3
$$

$x_s$表示决策中超过了实际需要的部分，影次常称它为剩余变量。$x_s\geq 0$

由上可见，既把函数从不等式转变成了等式，并且新添加的变量也是非负的，也不影响原来的目标函数。统一整个计算过程。

**$b_i$非负**

这个只需要将约束条件两端取反即可。

**$x_i$非负**

1. 若$x_i\leq 0$，则令$x'_i=-x_i$，此时$x'_i\geq 0$
2. 若$x_i$无约束，则令$x_i = x_i'-x_i'',x_i'\geq0,x_i''\geq0$

注意这里的变换要把变换也代入目标函数中

## 线性规划解的基本性质

对于线性规划问题来说，可行解实际上是由约束条件构成的线性方程组的解。并且还满足非负约束条件，即决策变量都取非负值。

我们假定线性规划问题的系数矩阵$A$的秩总等于其行数，$rank(A)=m$。这意味着系数矩阵$A$的各行是线性无关的，这也表明约束方程中的各个方程是相互独立的。

由于$A$的秩为$m$，所以其必有一个$m\times m$的子矩阵为$B$，其行列式不为零。

我们将$A$的任意一个这样的子矩阵$B$称为线性规划问题的一个基。

显然$B=[p_1,p_2,\cdots,p_m]$，其中的每一个为基向量，与基向量对应的变量$x_j$称为基变量，$x_B = [x_1,x_2,\cdots,x_m]^T$，其余称为非基变量，$x_N=[x_{m+1},x_{m+2},\cdots,x_n]^T$。

可以得到一个解$x=[x_B^T,x_N^T]^T=[x_1,x_2,\cdots,x_m,0,\cdots,0]^T$

称$x$为该约束方程的基解，其中$x_b=B^{-1}b$

对于满足非负约束条件的$x\geq 0$（所有分量都大于等于$0$）的基解称为可行解，对应的基称为可行基。

基可行解的非零分量个数小于$m$时，称为退化解。也就是说，不是所有基解都是可行解。

**定理1**

线性规划问题的基可行解是其可行域的顶点。

**定理2**

若可行域有界，线性规划问题的目标函数则一定在其可行域的极点上达到最优。

## 单纯形法

前面提到，可行解一定在凸集的顶点上，但是，这个顶点的数量很可能非常大。单纯形法就是在这些顶点中优化的算法

基本思路

1. 首先将线性规划问题化成标准形式
2. 求出初始基本可行解
3. 判断其是否为最优解
4. 如果不是最优，则迭代到其相邻的基本可行解，并再次检验

单纯形法把寻优的目标集中在所有基本可行解（即可行域顶点）中。从一个初始的基本可行解出发，寻找一条达到最优基本可行解的最佳路径。
