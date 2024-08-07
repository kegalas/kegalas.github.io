---
title: 概率论与数理统计整理
date: 2022-08-30T10:41:24+08:00
draft: false
tags:
  - 数学
  - 大学
  - 水文
description: 概率论与数理统计的学习笔记
categories: 数学
image: cover.jpg
mathjax: true
markup: goldmark
---

# 基本概念

## 随机试验E

针对随机现象的观察、记录、试验（广义）。

特点：

1. 可以在相同的条件下重复进行
2. 每次试验的可能结果不止一个，并且能事先明确试验的所有可能结果
3. 但一次试验前不能确定哪个结果会出现

## 样本空间$\Omega$

随机试验$E$的所有结果构成的集合为$E$的样本空间$\Omega$，记为$\Omega\{\omega\}$，每个结果$\omega$是$\Omega$中的一个元素，称为样本点。

## 频率

在相同的条件下，进行了$n$次试验，其中事件$A$发生的次数$n_A$称为事件$A$发生的频数，比值$n_A/n$称为事件$A$发生的频率，记为$f_n(A)$

## 概率

定义1: 事件A发生频率的稳定值$p$称为它的概率$P(A)$，即$P(A)=p$

定义2: 随机试验$E$，样本空间$\Omega$，对于$E$的每一事件$A$赋予一个实数，记为$P(A)$，如果集合函数满足以下条件，P(A)称为事件A的概率

1. 非负性: 对于每一个事件$A$，有$P(A)\geq0$
2. 规范性: 对于必然事件$\Omega$，有$P(\Omega)=1$
3. 可列可加性: 设$A_1,A_2,\cdots$是两两不相容的事件，即对于$A_iA_j=\empty,i\neq j(i,j=1,2,\cdots)$

$$
P(\bigcup^\infty_{i=1}A_i)=\sum^\infty_{i=1}P(A_i)
$$

**性质**

1. $P(\empty)=0$
2. 可列可加性（见上）
3. 若$A\subset B$，则$P(B-A)=P(B)-P(A)$
4. 任意事件$A$，$P(A)\leq 1$
5. 任意事件$A$，$P(\overline{A})=1-P(A)$
6. 加法公式，对任意事件$A,B$，$P(A\cup B)=P(A)+P(B)-P(AB)$，扩展情况同容斥原理。
7. 极限性：设$A_1\subset A_2\subset\cdots$，则$\lim_{n\to\infty}P(A_n)=P(\bigcup^{\infty}_{i=1}A_i)$；设$A_1\supset A_2\supset\cdots$，则$\lim_{n\to\infty}P(A_n)=P(\bigcap^{\infty}_{i=1}A_i)$


注意，没有以下性质

1. $P(A)=0$不能推出$A=\empty$
2. $P(A)=1$不能推出$A=\Omega$
3. $P(A)=P(B)$不能推出$A=B$

## 古典概率

### 定义

设一个试验有$N$个等可能的结果，而事件$E$恰包含其中的$M$个结果，则事件$E$的概率，记为$P(E)$，定义为

$$
P(E)=\frac{M}{N}
$$

### 一些有用的公式

1. n个相异物件取$r(1\leq r\leq n)$个的不同排列总数

$$
P^n_r=n(n-1)(n-2)\cdots(n-r+1)
$$

2. n个相异物体取$r(1\leq r\leq n)$个的不同组合总数

$$
C^n_r = \frac{P^n_r}{r!}=\frac{n!}{r!(n-r)!}=\frac{n(n-1)(n-2)\cdots(n-r+1)}{r!}=\binom{n}{r}
$$

另外，只要$r$为非负整数，不论$n$为任何实数，都有意义。例如

$$
\binom{-1}{r} = \frac{(-1)(-2)\cdots(-r)}{r!}=(-1)^r
$$

3. 二项式系数

$$
(a+b)^n=\sum_{i=0}^n\binom{n}{i}a^ib^{n-i}
$$

令$a=b=1$

$$
\binom{n}{0}+\binom{n}{1}+\cdots+\binom{n}{n}=2^n
$$

令$a=-1,b=1$

$$
\binom{n}{0}-\binom{n}{1}+\binom{n}{2}+\cdots+(-1)^n\binom{n}{n}=0
$$

还有

$$
\binom{m+n}{k}=\sum_{i=0}^k\binom{m}{i}\binom{n}{k-i}
$$

4. $n$个相异物件分成$k$堆，各堆物件数分别为$r_1,\cdots,r_k$的分法是

$$
\frac{n!}{r_1!\cdots r_n!}
$$

以上是有序堆的情况，无序堆时

$$
\frac{n!}{P^k_k(r_1!\cdots r_n!)}
$$

## 几何概率

古典概率的样本空间为有限集，如果样本空间是无限集，就应该使用几何概率。

1. 直线上的几何概率：设线段$l$是$L$的一部分，向$L$上随机投一点，投中$l$的概率为

$$
P=\frac{l的长度}{L的长度}
$$

2. 平面、体积上的概率：类似于直线上，通过总体的面积或体积，以及事件所代表的面积或体积来计算。

## 事件的运算

### 蕴含、包含和相等

在同一试验下的两个事件$A,B$，若$A$发生时$B$必发生，则称$A$蕴含$B$，或者说$B$包含$A$，记为$A\subset B$，若它们互相蕴含，则称为相等，记为$A=B$

### 互斥和对立

若$A,B$不能在同一次试验中都发生（但可以都不发生），则称他们是互斥的（不相容的）。

对立事件是互斥事件的一种特殊情况。若$A$为一事件，则事件

$$
B=\{A不发生\}
$$

称为$A$的对立事件，记为$\bar{A}$

### 事件的和

$$
C=\{A发生，或B发生\}=\{A，B至少发生一个\}=A+B=A\cup B
$$

### 概率的加法定理

若干个互斥事件之和的概率，等于各事件的概率之和

$$
P(A_1+A_2+\cdots)=P(A_1)+P(A_2)+\cdots
$$

若不是两两互斥的事件，则要考虑用容斥原理来计算。

上式能推出

$$
P(\bar A)=1-P(A)
$$

### 事件的积

$$
C=\{A,B都发生\}=AB=A\cap B
$$

### 事件的差

$$
C=\{A发生，B不发生\}=A-B
$$

显然有

$$
A-B = A\bar B
$$

### 两个重要公式

$$
\overline{A_1A_2\cdots A_n}=\sum_{i=1}^n\bar{A_i}
$$

$$
\overline{A_1+A_2+\cdots+A_n}=\prod_{i=1}^n\bar{A_i}
$$

## 条件概率

设有两个事件$A,B$，而$P(B)\neq0$。则“在给定$B$发生的条件下$A$的条件概率”，记为$P(A|B)$，定义为

$$
P(A|B)=P(AB)/P(B)
$$

但并不一定要用这个公式去计算，有时直接用加入了条件的情况去算会更为方便。

## 事件的独立性、概率乘法定理

若$P(A)=P(A|B)$，则$B$的发生与否对$A$发生的可能性毫无影响。这时，在概率论上称$A,B$两事件独立。

同时就有

$$
P(AB) = P(A)P(B)
$$

若两事件$A,B$满足上式，则称$A,B$独立。反过来说独立的两个事件就有上式，也称为概率的乘法定理。

将其推广，设$A_1,A_2,\cdots$为有限或无限个事件。如果从其中任意取出有限个$A_{i_1},A_{i_2},\cdots,A_{i_m}$，都有

$$
P(A_{i_1}A_{i_2}\cdots A_{i_m})=P(A_{i_1})P(A_{i_2})\cdots P(A_{i_m})
$$

则称$A_1,A_2,\cdots$互相独立。互相独立的几个事件则具有以上的乘积性质。

等价的来说也有

$$
P(A_{i_1}|A_{i_2}\cdots A_{i_m})=P(A_{i_1})
$$

可以推出：独立事件的任一部分也独立。

还有：若一列事件相互独立，则将其中任一部分改为对立事件时，所有事件仍然相互独立。

## 全概率公式

设$B_1,B_2,\cdots$为有限或无限个事件，它们两两互斥且在每次试验中至少发生一个。即

$$
B_iB_j=\empty(i\neq j)
$$

$$
B_1+B_2+\cdots = \Omega（必然事件）
$$

有时，这样的一组事件称为完备事件群。

现有任一事件$A$，有$A=A\Omega=AB_1+AB_2+\cdots$。而$B_1,B_2,\cdots$两两互斥，则$AB_1,AB_2,\cdots$也两两互斥，固有

$$
P(A) = P(AB_1)+P(AB_2)+\cdots
$$

再由条件概率$P(AB_i)=P(B_i)P(A|B_i)$，得

$$
P(A) = P(B_1)P(A|B_1) + P(B_2)P(A|B_2) + \cdots
$$

上式称为全概率公式。

## 贝叶斯公式

由全概率公式，有

$$
P(B_i|A)=P(AB_i)/P(A)\\=P(B_i)P(A|B_i)/\sum_jP(B_j)P(A|B_j)
$$

# 随机变量及其分布

## 随机变量的概念

随机变量就是其值随机会而定的变量。例如从一大批产品中随机抽出100个，其中所含的废品数为$X$。这个$X$就是一个随机变量。

随机变量区分为两大类

1. 离散型随机变量。其特征是只能取有限个值，或者虽然能取无限个，但可以一个一个地排列出来（类似于可数无限）。
2. 连续性随机变量。不仅是无穷多的，而且类似于不可数无限，充满一个区间。

## 离散型随机变量

**定义1**

设$X$为离散型随机变量，其全部可能值为$\{a_1,a_2,\cdots\}$，则

$$
p_i=P(X=a_i)\quad(i=1,2,\cdots)
$$

称为$X$的概率函数

显然有

$$
p_i\geq 0,\quad p_1+p_2+\cdots = 1
$$

**定义2**

设$X$为一随机变量，则函数

$$
P(X\leq x) = F(x)\quad(-\infty< x<\infty)
$$

称为$X$的分布函数。这里不限制$X$是离散型的。

$$
F(x) = P(X\leq x) = \sum_{\{i|a_i\leq x\}}p_i
$$

分布函数具有如下性质

1. $F(x)$是单调非降的：$x_1\leq x_2$时，有$F(x_1)\leq F(x_2)$
2. $x\to \infty$时，$F(x)\to 1$。$x\to -\infty$时，$F(x)\to 0$

## 连续型随机变量

与离散型的有限或可数无限不同，这是一种充满整个区间的随机变量，或者说不可数无限。

刻画连续型随机变量可以用之前提到的概率分布函数，但是更常用概率密度函数。

### 概率密度函数

设连续型随机变量$X$有概率分布函数$F(x)$，则$F(x)$的导数$f(x)=F'(x)$称为$X$的概率密度函数。

具有如下性质

1. $f(x)\geq 0$
2. $\int^\infty_{-\infty}f(x)dx=1$
3. 对于任何常数$a< b$，有

$$
P(a\leq X\leq b)=F(b)-F(a) = \int^b_af(x)dx
$$

## 多维随机变量（随机向量）

### 离散型随机向量的分布

一般的，设$X=(X_1,X_2,\cdots,X_n)$为一个$n$维向量，其每个分量，即$X_1,\cdots,X_n$都是一维随机变量，则称$X$是一个$n$维随机向量或$n$维随机变量。

如果一个随机向量$X$，其每一个分量$X_i$都是一维离散型随机变量，则称$X$是离散型的。

以$\{a_{i1},a_{i2},\cdots\}$记$X_i$的全部可能值，则事件$\{X_1=a_{1j_1}, X_2=a_{2j_2},\cdots,X_n=a_{nj_n}\}$的概率

$$
p(j_1,j_2,\cdots,j_n)=P(X_1=a_{1j_1}, X_2=a_{2j_2},\cdots,X_n=a_{nj_n})
$$

称为随机向量$X$的概率函数或概率分布，其应该满足

$$
p(j_1,j_2,\cdots,j_n)\geq 0,\sum_{j_n}\cdots\sum_{j_2}\sum_{j_1}p(j_1,j_2,\cdots,j_n) = 1
$$

### 连续型随机向量的分布

设$X$是一个$n$维随机向量，其取值可视为$n$维欧式空间$\mathbb{R} ^n$中的一个点。如果$X$的全部取值能充满$\mathbb{R} ^n$中某一区域，则称它是连续型的。

若$f(x_1,\cdots,x_n)$是定义在$\mathbb{R} ^n$上的非负函数，使对$\mathbb{R} ^n$中的任何集合$A$，有

$$
P(X\in A) = \int\cdots\int f(x_1,\cdots,x_n)dx_1\cdots dx_n
$$

则称$f$是$X$的概率密度函数

如果把$A$取成$\mathbb{R} ^n$，则有

$$
\int^\infty_{-\infty}\cdots\int^\infty_{-\infty} f(x_1,\cdots,x_n)dx_1\cdots dx_n = 1
$$

对于二维情况

$$
F(x,y) = \int^y_{-\infty}\int^x_{-\infty}f(u,v)dudv
$$

$$
\frac{\partial^2 F(x,y)}{\partial x\partial y}=f(x,y)
$$

在$\Delta x,\Delta y$很小时，有$P(x< X< x+\Delta x,y< Y< y+\Delta y)\approx f(x,y)\Delta x\Delta y$

若$G$为$xOy$内的任意区域，点$(x,y)$落在$G$内的概率为

$$
P\{(X,Y)\in G\}=\iint_G f(x,y)dxdy
$$

与一维的情况一样，也可以用概率分布函数去描述多维随机向量的概率分布，其定义为

$$
F(x_1,x_2,\cdots,x_n)=P(X_1\leq x_1, X_2\leq x_2,\cdots,X_n\leq x_n)
$$

其基本性质，以二维为例，有

1. $F(x,y)$是$x,y$的不减函数
2. $0\leq F(x,y)\leq 1,F(-\infty,-\infty) = 0,F(\infty,\infty)=1$

对于任意$x$，$F(x,-\infty)=0$，对于任意$y$，$F(-\infty,y)=0$

3. $F(x,y)$关于$x,y$均为右连续，$F(x+0,y) = F(x,y)$，$F(x,y+0) = F(x,y)$
4. 若$x_1< x_2,y_1< y_2$，则$F(x_2,y_2)+F(x_1,y_1)-F(x_2,y_1)-F(x_1,y_2)\geq 0$

另外，有

$$
f(x,y) = \frac{\partial^2F(x,y)}{\partial x\partial y}
$$

### 边缘分布

设$X=(X_1,\cdots,X_n)$为一个$n$维随机向量。$X$有一定的分布$F$，这是一个$n$维分布，因为$X$的每个分量$X_i$都是一维随机变量，故它们都有各自的分布$F_i$，这些都是一维分布，称为随机向量$X$或其分布$F$的边缘分布。边缘分布完全由原分布$F$确定。

例如，在离散型的情况下，边缘分布$P(X_1=a_{1k})$就等于

$$
P(X_1=a_{1k})=\sum_{j_2,\cdots,j_n}p(k,j_2,\cdots,j_n)
$$

对于连续型的情况，例如求$f_1(x_1)$

$$
f_1(x_1) = \int^\infty_{-\infty}\cdots \int^\infty_{-\infty}f(x_1,x_2,\cdots,x_n)dx_2\cdots dx_n
$$

当然，如果题目有要求变量的范围，则积分上下限要改成相应的值。

直观上的理解，就是将其他变量的所有情况的概率全部加起来。

### 联合分布

边缘分布指的是随机向量中的一个分量的分布。

而相对应的联合分布，就是强调$(X_1,\cdots,X_n)$的分布是把$X_1,\cdots,X_n$作为一个有联系的整体来考虑的。

当然有些时候边缘分布也可以指众多分量中几个分量的分布。

## 条件概率分布

概念同前面提到的条件概率。

### 离散型

同之前计算条件概率相同，只不过通常此时要用到边缘分布等概念。

例如设

$$
p_{ij}=P(X_1=a_i,X_2=b_j)
$$

则

$$
P(X_1=a_i|X_2=b_j) = P(X_1=a_i,X_2=b_j)/P(X_2=b_j) = p_{ij}/P(X_2=b_j)
$$

### 连续型

以二维情况为例

$$
P(X_1\leq x_1|a\leq X_2\leq b)=P(X_1\leq x_1,a\leq X_2\leq b)/P(a\leq X_2\leq b)
$$

写成积分形式

$$
=\int^{x_1}_{-\infty}dt_1\int^b_af(t_1,t_2)dt_2\bigg/\int^b_af_2(t_2)dt_2
$$

$a=b$的情况下，可以通过极限求出

$$
f_1(x_1|x_2) = f(x_1,x_2)/f_2(x_2)
$$

可改写为

$$
f(x_1,x_2) = f_1(x_1|x_2)f_2(x_2)
$$

可以推广到

$$
f(x_1,\cdots,x_n) = g(x_1,\cdots,x_k)h(x_{k+1},\cdots,x_n|x_1,\cdots,x_k)
$$

## 随机变量的独立性

**定义1**

设$n$维随机向量$(X_1,\cdots,X_n)$的联合密度函数为$f(x_1,x_2,\cdots,x_n)$，而$X_i$的边缘密度函数为$f_i(x_i)$，如果

$$
f(x_1,\cdots,x_n) = f_1(x_1)\cdots f_n(x_n)
$$

则称随机变量$X_1,\cdots,X_n$相互独立。

**定理1**

如果连续变量$X_1,\cdots,X_n$相互独立，则对任何$a_i< b_i$，则由下式定义的$n$个事件也独立

$$
A_1=\{a_1\leq X_1\leq b_1\},\cdots,A_n=\{a_n\leq X_n\leq b_n\}
$$

反之，若对任何$a_i< b_i$，事件$A_1,\cdots,A_n$独立，则变量$X_1,\cdots,X_n$独立。

**定理2**

若随机向量$(X_1,\cdots,X_n)$的概率密度函数$f(x_1,x_2,\cdots,x_n)$可表示为$n$个函数$g_1,\cdots,g_n$之积，其中$g_i$只依赖于$x_i$，即

$$
f(x_1,x_2,\cdots,x_n) = g_1(x_1)\cdots g_n(x_n)
$$

则$X_1,\cdots,X_n$相互独立，且$X_i$的边缘密度函数$f_i(x_i)$与$g_i(x_i)$只相差一个常数因子。

**定理3**

若$X_1,\cdots,X_n$相互独立，而

$$
Y_1=g_1(X_1,\cdots,X_m), Y_2 = g_2(X_{m+1},\cdots,X_n)
$$

则$Y_1,Y_2$独立。

**定义2**

设$X_1,\cdots,X_n$是离散型随机变量，若对任何常数$a_1,\cdots,a_n$都有

$$
P(X_1=a_1,\cdots,X_n=a_n) = P(X_1=a_1)\cdots P(X_n=a_n)
$$

则称$X_1,\cdots, X_n$相互独立。

## 随机变量的函数的概率分布

在理论和应用上，经常碰到这种情况：已知某个或某些随机变量$X_1,\cdots,X_n$的分布，现另有一些随机变量$Y_1,\cdots,Y_m$，它们都是$X_1,\cdots,X_n$的函数：

$$
Y_i=g_i(X_1,\cdots,X_n)
$$

要求$(Y_1,\cdots,Y_m)$的概率分布。

### 离散型分布的情况

离散分布的情况比较简单，只需要列举$Y_i=y_i$时的所有$X$的可能就可以了。

### 连续型分布的情况的一般讨论

先考虑一个变量的情况，设$X$有密度函数$f(x)$. 设$Y=g(x)$。$g$严格上升或下降。所以$g'$存在，反函数$X=g^{-1}(Y)$存在，且$(g^{-1})'$也存在。

则$Y$的密度函数为

$$
f_Y(y)=f(g^{-1}(y))|(g^{-1}(y))'|
$$

现在考虑多个变量，以两个变量为例，设$(X_1,X_2)$的密度函数为$f(x_1,x_2)$，以及

$$
Y_1=g_1(X_1,X_2),\quad Y_2=g_2(X_1,X_2)
$$

要求$f_Y(y_1,y_2)$，假定上式是一一对应变换，则有逆变换

$$
X_1=g^{-1}_1(Y_1,Y_2),\quad X_2=g^{-1}_2(Y_1,Y_2)
$$

假设$g_1,g_2$都有一阶连续偏导数，则其反函数也有一阶连续偏导数，且雅克比行列式

$$
J(y_1,y_2)=
\begin{vmatrix}
 \partial g^{-1}_1/\partial y_1 & \partial g^{-1}_1/\partial y_2\\
 \partial g^{-1}_2/\partial y_1 & \partial g^{-1}_2/\partial y_2
\end{vmatrix}
$$

不为$0$. 有

$$
f_Y(y_1,y_2) = f(g^{-1}_1(y_1,y_2),g^{-1}_2(y_1,y_2))|J(y_1,y_2)|
$$

### 随机变量和的密度函数

设$(X_1,X_2)$的联合密度函数为$f(x_1,x_2)$，要求

$$
Y=X_1+X_2
$$

的密度函数。

有

$$
f_Y(y) = \int^\infty_{-\infty}f(x_1,y-x_1)dx_1=\int^\infty_{-\infty}f(x,y-x)dx
$$

或者作变量代换也有

$$
f_Y(y) = \int^\infty_{-\infty}f(y-x,x)dx
$$

如果$X_1,X_2$独立，则$f(x_1,x_2)=f_1(x_1)f_2(x_2)$，有

$$
f_Y(y) = \int^\infty_{-\infty}f_1(x)f_2(y-x)dx = \int^\infty_{-\infty}f_1(y-x)f_2(x)dx
$$

### 随机变量商的密度函数

设$(X_1,X_2)$有密度函数$f(x_1,x_2)$，$Y=X_2/X_1$，要求$Y$的密度函数

$$
f_Y(y) = \int^\infty_{-\infty} |x_1|f(x_1,x_1y)dx_1
$$

若$X_1,X_2$独立，则

$$
f_Y(y) = \int^\infty_{-\infty} |x_1|f_1(x_1)f_2(x_1y)dx_1
$$

### 随机变量积的密度函数

设$(X_1,X_2)$有密度函数$f(x_1,x_2)$，$Y=X_1X_2$，要求$Y$的密度函数

$$
f_Y(y) = \int^\infty_{-\infty} \frac{1}{|x_1|}f(x_1,\frac{y}{x_1})dx_1
$$

若$X_1,X_2$独立，则

$$
f_Y(y) = \int^\infty_{-\infty} \frac{1}{|x_1|}f_1(x_1)f_2(\frac{y}{x_1})dx_1
$$

## 常见的分布

### 0-1分布

设某个事件$A$在一次试验中发生的概率为$p$，重复试验$1$次，以$X$记$A$在$1$次试验中发生的次数，则

$$
P(X=i)=p^i(1-p)^{1-i}\quad (i=0,1)
$$

#### 数学期望

$$
p
$$

#### 方差

$$
p(1-p)
$$

### 二项分布

设某个事件$A$在一次试验中发生的概率为$p$，重复试验$n$次，以$X$记$A$在$n$次试验中发生的次数，则

$$
p_i=b(i;n,p)=\binom{n}{i}p^i(1-p)^{n-i}\quad (n=0,1,\cdots,n)
$$

服从有两个条件

1. 各次试验的条件是稳定的
2. 各次试验的独立性

如果抽检后放回，则每次抽出废品的概率是相等的。如果抽检后不放回，则废品率发生了变化，试验条件不稳定，不符合二项分布。但是如果总数远大于抽出的数量，则不放回也几乎不影响，可以近似地作为二项分布。

通常也会记作，$B(n,p)$

#### 数学期望

$$
np
$$

#### 方差

$$
np(1-p)
$$

### 泊松分布

若随机变量$X$的可能取值为$0,1,2,\cdots$，且概率分布为

$$
P(X=i)=e^{-\lambda}\lambda^i/i!
$$

则称$X$服从泊松分布。记为$X\sim P(\lambda)$。$\lambda>0$是某一常数。式子右边对$i=0,1,\cdots$求和的结果为$1$。可以从$e^\lambda=\sum^\infty_{i=0}\lambda^i/i!$得出。

这个分布多出现在当$X$表示在一定时间或空间内出现的事件个数的场合。比如一定时间内某交通路口所发生的事故个数。

泊松分布可以看做二项分布的极限。若$X\sim B(n,p)$中$n$很大，$p$很小，而$np=\lambda$不太大时，$X$的分布接近与泊松分布$P(\lambda)$。推导如下：

$$
P(X=i)=\binom{n}{i}\left(\frac{\lambda}{n}\right)^i\left(1-\frac{\lambda}{n}\right)^{n-i}
$$

当$n\to\infty$时

$$
\binom{n}{i}/n^i\to 1/i!,\quad \left(1-\frac{\lambda}{n}\right)^{n-i}\to e^{-\lambda}
$$

则有

$$
P(X=i)=e^{-\lambda}\lambda^i/i!
$$

显然通常我们不会遇到$n$无限的情况，只会遇到$n$较大的情况，所以只能说接近泊松分布。

#### 数学期望

$$
\lambda
$$

我们可以证明$X$服从泊松分布时

$$
E(X) = \sum^\infty_{i=0}i\frac{\lambda^i}{i!}e^{-\lambda}=\lambda
$$

#### 方差

$$
\lambda
$$

### 超几何分布

以$X$记从含$M$个废品的$N$个产品中随机抽出$n$个里面所含有的废品数，$X$的分布为

$$
P(X=m)=\binom{M}{m}\binom{N-M}{n-m}\bigg/\binom{N}{n}
$$

这是一种抽检不放回的试验，如果$N$特别大，$n$不够大时，放回与不放回差别不大。可以当作二项分布来看。或者说，若$X$服从超几何分布，则当$n$固定时，$M/N=p$固定；$N\to \infty$时，$X$近似服从二项分布$B(n,p)$。

#### 数学期望

$$
\frac{nM}{N}
$$

#### 方差

$$
\frac{nM(N-M)(N-n)}{N^2(N-1)}
$$

### 负二项分布

先指定一个自然数$r$，一个一个地从一批产品中抽样检查，直到发现第$r$个废品，以$X$记此时已经抽出的合格产品个数。

显然$\{X=i\}$这个事件发生时，需要满足

1. 在前$i+r-1$次抽取中，正好有$r-1$个废品
2. 第$i+r$次抽出废品

所以有

$$
P(X=i) = b(r-1;i+r-1,p)p
$$

$$
=\binom{i+r-1}{r-1}p^r(1-p)^i
$$

负二项分布的名称来源于

$$
(1-x)^{-r}=\sum_{i=0}^{\infty}\binom{-r}{i}(-x)^i=\sum_{i=0}^{\infty}\binom{i+r-1}{i}x^i
$$

$$
=\sum_{i=0}^{\infty}\binom{i+r-1}{r-1}x^i
$$

其中令$x=1-p$，两边乘以$p^r$，得

$$
1 = p^r[1-(1-p)]^{-r}=\sum_{i=0}^{\infty}\binom{i+r-1}{r-1}p^r(1-p)^i
$$

### 几何分布

当负二项分布中的$r=1$时，有

$$
P(X=i) = p(1-p)^i
$$


#### 数学期望

$$
\frac{1}{p}
$$

#### 方差

$$
\frac{1-p}{p^2}
$$

### 多项分布

设$A_1,A_2,\cdots,A_n$是某一试验之下的完备事件群，即事件$A_1,\cdots,A_n$两两互斥，其和为必然事件。分别以$p_1,p_2,\cdots,p_n$记每个事件对应的概率。

现将试验独立重复$N$次，而以$X_i$记载着$N$次试验中事件$A_i$出现的次数。$X$的概率分布就叫做多项分布，有时记为$M(N;p_1,\cdots,p_n)$。其概率有

$$
P(X_1=k_1,X_2=k_2,\cdots,X_n=k_n)=\frac{N!}{k_1!k_2!\cdots k_n!}p_1^{k_1}p_2^{k_2}\cdots p_n^{k_n}
$$

### 正态分布

如果一个随机变量具有概率密度函数

$$
f(x)=(\sqrt{2\pi}\sigma)^{-1}e^{-\frac{(x-\mu)^2}{2\sigma^2}}\quad (-\infty < x < + \infty )
$$

则称$X$为正态随机变量，并记为$X\sim N(\mu,\sigma^2)$。$\mu,\sigma^2$是常数，$\mu\in R,0<\sigma^2<\infty$，它们称为这个分布的参数。由后续的方差一节，$Var(X)=\sigma^2$，就是分布的方差。

正态分布的概率密度函数的图像关于$x=\mu$对称，中间高两边低。

当$\mu=0,\sigma^2=1$时，称为标准正态分布$N(0,1)$

$$
f(x)=e^{-x^2/2}/\sqrt{2\pi}
$$

通常$N(0,1)$的密度函数和分布函数也会记为$\varphi(x)$和$\varPhi(x)$。

任意一个正态分布都可以转化为标准正态分布，便于查表。

$$
if\quad X\sim N(\mu,\sigma^2),then\quad Y=(X-\mu)/\sigma\sim N(0,1)
$$

例如$X\sim N(1.5,2^2)$，要计算$P(-1\leq X\leq 2)$，则因$(X-1.5)/2\sim N(0,1)$，有

$$
P(-1\leq X\leq 2) = P\left(\frac{-1-1.5}{2}\leq\frac{X-1.5}{2}\leq\frac{2-1.5}{2}\right)
$$

$$
=P(-1.25\leq(X-1.5)/2\leq 0.25)
$$

$$
=\varPhi(0.25)-\varPhi(-1.25)
$$

可以查表代入。但是通常只有非负数的值，对于负数$x$，有

$$
\varPhi(x) = 1-\varPhi(-x)
$$

#### 正态分布的线性组合性质

对于正态分布，设$X_1,X_2$分别服从正态分布$N(\mu_1,\sigma_1^2),N(\mu_2,\sigma_2^2)$，对于$Y=X_1+X_2$

1. $X_1,X_2$独立，则$Y\sim N(\mu_1+\mu_2, \sigma_1^2+\sigma_2^2)$。并且反过来说$Y$符合正态分布，而$Y$可以表示为两个独立变量的和$Y=X_1+X_2$，则$X_1,X_2$必须服从正态分布。
2. $X_1,X_2$不独立，但其联合分布为二维正态分布$N(\mu_1,\mu_2, \sigma_1^2,\sigma_2^2,\rho)$，则$Y$仍然服从正态分布$Y\sim N(\mu_1+\mu_2, \sigma_1^2+\sigma_2^2+2\rho\sigma_1\sigma_2)$

同样可以推广到多维情况，$X_1,\cdots,X_n$相互独立，分别服从$N(\mu_1,\sigma_1^2),\cdots,N(\mu_n,\sigma_n^2)$，则$X_1+\cdots+X_n$服从$N(\mu_1+\cdots+\mu_n,\sigma_1^2+\cdots+\sigma_n^2)$

对于不全为$0$的常数$C_0,C_1,\cdots,C_n$，线性组合

$$
C_0+C_1X_1+\cdots+C_nX_n\sim N(C_0+C_1\mu_1+\cdots+C_n\mu_n,C_1^2\sigma_1^2+\cdots+C_n^2\sigma_n^2)
$$


#### 数学期望

$$
\mu
$$

#### 方差

$$
\sigma^2
$$

#### 上$\alpha$分位点 

标准正态分布的上$\alpha$分位点查表可知，具有一个性质为$w_{1-\alpha}=-w_\alpha$。对于非标准的则和中位数对称。

### 二维正态分布

$$
f(x_1,x_2)=(2\pi\sigma_1\sigma_2\sqrt{1-\rho^2})^{-1}\exp\left[-\frac{1}{2(1-\rho^2)}\left(\frac{(x_1-a)^2}{\sigma_1^2}-\frac{2\rho(x_1-a)(x_2-b)}{\sigma_1\sigma_2}+\frac{(x_2-b)^2}{\sigma_2^2}\right)\right]
$$

常把这个分布记为$N(a,b,\sigma_1^2,\sigma_2^2,\rho)$

### n维正态随机变量

**定义1**

设$(X_1,X_2)$的四个二阶中心矩都存在，把

$$
\begin{bmatrix}
 Cov(X_1) & Corr(X_1,X_2) \\
 Corr(X_2,X_1) & Cov(X_2)
\end{bmatrix}
$$

称为随机变量$(X_1,X_2)$的协方差矩阵。

扩展到$n$维就是

$$
\begin{bmatrix}
 Cov(X_1) & Corr(X_1,X_2) & \cdots & Corr(X_1,X_n)\\
 Corr(X_2,X_1) & Cov(X_2) & \cdots & Corr(X_2,X_n)\\
 \vdots & \vdots &  & \vdots\\
 Corr(X_n,X_1) & Corr(X_n,X_2) & \cdots & Cov(X_n)
\end{bmatrix}
$$

当然所有的协方差必须存在。

对于二维正态分布

$$
f(x_1,x_2)=\\
(2\pi\sigma_1\sigma_2\sqrt{1-\rho^2})^{-1}\exp\left[-\frac{1}{2(1-\rho^2)}\left(\frac{(x_1-\mu_1)^2}{\sigma_1^2}-\frac{2\rho(x_1-\mu_1)(x_2-\mu_2)}{\sigma_1\sigma_2}+
\frac{(x_2-\mu_2)^2}{\sigma_2^2}\right)\right]
$$

其中$\rho$是$X,Y$的相关系数

所以$(X_1,X_2)$的协方差矩阵是

$$
B = \begin{bmatrix}
 \sigma_1^2 & \rho\sigma_1\sigma_2 \\
 \rho\sigma_1\sigma_2 & \sigma_2^2
\end{bmatrix}
$$

定义

$$
X=\begin{bmatrix}
 X_1 \\
 X_2 
\end{bmatrix},
\mu = \begin{bmatrix}
 \mu_1 \\
 \mu_2 
\end{bmatrix}
$$

则$f(x_1,x_2)$也可以写为

$$
f(x_1,x_2)=(2\pi|B|^{1/2})exp\bigg[-\frac{1}{2}(X-\mu)^TB^{-1}(X-\mu)\bigg]
$$

扩展到$n$维，则有

$$
f(x_1,x_2,\cdots,x_n)=[(2\pi)^{n/2}|B|^{1/2}]exp\bigg[-\frac{1}{2}(X-\mu)^TB^{-1}(X-\mu)\bigg]
$$

**n维正态变量的性质**

1. $n$维正态随机变量$(X_1,X_2,\cdots,X_n)$的每一个分量$X_i$都是正态随机变量；反之若$X_1,X_2,\cdots,X_n$都是正态随机变量且相互独立，则$(X_1,X_2,\cdots,X_n)$是$n$维正态随机变量
2. $(X_1,X_2,\cdots,X_n)$是$n$维正态随机变量的充要条件是$X_1,X_2,\cdots,X_n$的任意线性组合$l_1X_1,l_2X_2,\cdots,l_nX_n$服从一维正态分布（其中$l_1,l_2,\cdots,l_n$不全为$0$）
3. 若$(X_1,X_2,\cdots,X_n)$服从$n$维正态分布，设$Y_1,Y_2,\cdots,Y_k$是$X_i$的线性变换，则$(Y_1,Y_2,\cdots,Y_k)$也服从$k$维正态分布
4. 若$(X_1,X_2,\cdots,X_n)$是$n$维正态随机变量，$m < n$，则$(X_1,X_2,\cdots,X_n)$的任意$m$个分量是$m$维正态随机变量
5. 设$(X_1,X_2,\cdots,X_n)$是$n$维正态随机变量，则“$X_1,X_2,\cdots,X_n$相互独立”与“$X_1,X_2,\cdots,X_n$两两不相关”等价。


### 指数分布

若随机变量$X$有概率密度函数

$$
f(x)=\left\{\begin{matrix}
\lambda e^{-\lambda x},\quad x>0\\
0,\quad x\leq 0
\end{matrix}\right.
$$

则$X$服从指数分布，其中$\lambda>0$是参数。通常记作$E(\lambda)$

其概率分布函数为

$$
F(x)=\left\{\begin{matrix}
1-e^{-\lambda x},\quad x>0\\
0,\quad x\leq 0
\end{matrix}\right.
$$

#### 数学期望

$$
\frac{1}{\lambda}
$$

#### 方差

$$
\frac{1}{\lambda^2}
$$

### 威布尔分布

满足

$$
f(x)=\left\{\begin{matrix}
\lambda\alpha x^{\alpha-1} e^{-\lambda x^\alpha},\quad x>0\\
0,\quad x\leq 0
\end{matrix}\right.
$$

其概率分布函数是

$$
F(x)=\left\{\begin{matrix}
1-e^{-\lambda x^\alpha},\quad x>0\\
0,\quad x\leq 0
\end{matrix}\right.
$$

可见，指数分布是威布尔分布的一个特例。

### 均匀分布

满足

$$
f(x)=\left\{\begin{matrix}
1/(b-a),\quad a\leq x\leq b\\
0,\quad else
\end{matrix}\right.
$$

其概率分布函数是

$$
F(x)=\left\{\begin{align*}
&0, &&x\leq a\\
&(x-a)/(b-a), &&a < x < b\\
&1, &&x\geq b
\end{align*}\right.
$$

常记作$R(a,b),U(a,b)$

#### 数学期望

$$
\frac{a+b}{2}
$$

#### 方差

$$
\frac{(b-a)^2}{12}
$$

### 二维随机向量的均匀分布

$$
f(x_1,x_2)=
\left\{\begin{matrix}
1/[(b-a)(d-c)],\quad a\leq x_1\leq b, c\leq x_2\leq d\\
0,\quad\quad\quad\quad\quad\quad\quad\quad\quad other\quad\quad\quad\quad\quad\quad
\end{matrix}\right.
$$

以上是矩形情况的密度函数

如果扩展到任意形状的图形，只要求出来其面积，那么密度函数在图形内部就是$1/S$，图形外部是$0$

### 最大值分布

设$M=max\{X,Y\}$，$X,Y$相互独立，则

$$
F_{max}(z)=P(M\leq z)=P(X\leq z, Y\leq z)=P(X\leq z)P(Y\leq z)
$$

$$
F_{max}(z) = F_X(z)F_Y(z)
$$

扩展到多维情况

$$
F_{max}(z) = F_{X_1}(z)F_{X_2}(z)\cdots F_{X_n}(z)
$$

### 最小值分布

设$N=min\{X,Y\}$，$X,Y$相互独立，则

$$
F_{min}(z)=P(N\leq z)=1-P(N>z)=1-P(X>z, Y>z)=1-P(X>z)P(Y>z)
$$

$$
F_{min}(z) = 1-[1-F_X(z)][1-F_Y(z)]
$$

扩展到多维情况

$$
F_{min}(z) = 1-[1-F_{X_1}(z)][1-F_{X_2}(z)]\cdots [1-F_{X_n}(z)]
$$

### 卡方分布

**$\Gamma$函数**

通过积分

$$
\Gamma(x) = \int^\infty_0 e^{-t}t^{x-1}dt(x>0)
$$

来定义

**$\Beta$（Beta）函数**

通过积分

$$
\Gamma(x) = \int^1_0 t^{x-1}(1-t)^{y-1}dt(x>0
,y>0)
$$

来定义

**$\Gamma$函数的性质**

1. $\Gamma(1)=1$
2. $\Gamma(1/2)=\sqrt{\pi}$
3. $\Gamma(x+1)=x\Gamma(x)$

因此可知，$n$为正整数时

$$
\Gamma(n) = (n-1)!
$$

$n$为正奇数时

$$
\Gamma(n/2) = 1\cdot 3\cdot 5\cdots(n-2)2^{-(n-1)/2}\sqrt{\pi}
$$

其中$\Gamma$与$\Beta$函数之间有重要的关系式

$$
\Beta(x,y)=\Gamma(x)\Gamma(y)/\Gamma(x+y)
$$

**卡方分布**

由$\Gamma$函数的定义可知，若$n>0$，则函数

$$
k_n(x)=\left\{\begin{align*}
&\frac{1}{\Gamma(n/2)2^{n/2}}e^{-x/2}x^{(n-2)/2} &,\quad x>0\\
&0 &,\quad x\leq 0
\end{align*}\right.
$$

是概率密度函数。它称为“自由度为$n$的皮尔逊卡方密度”，常记为$\mathcal{X}{}_n^2$

**定理1**

若$X_1,\cdots,X_n$相互独立，都服从正态分布$N(0,1)$，则$Y=X_1^2+\cdots+X_n^2$服从自由度为$n$的卡方分布$\mathcal{X}{}_n^2$

卡方分布有如下性质

1. 设$X_1,X_2$独立，$X_1\sim \mathcal{X}_m^2,X_2\sim \mathcal{X}_n^2$，则$X_1+X_2\sim \mathcal{X}_{m+n}^2$
2. 若$X_1,\cdots,X_n$独立，且都服从指数分布，则

$$
X=2\lambda(X_1+\cdots+X_n)\sim \mathcal{X}_{2n}^2
$$

**期望**

若$X\sim\mathcal{X}_n^2$，$E(X)=n$

**方差**

若$X\sim\mathcal{X}_n^2$，$D(X)=2n$

**上$\alpha$分位点**

可由查表得到。有一个性质，当$n$充分大，如$n>40$时，则其上$\alpha$分位点约为

$$
\dfrac{1}{2}\bigg(w_\alpha+\sqrt{2n-1}\bigg)^2
$$

其中$w_\alpha$是标准正态分布的上$\alpha$分位点

### t分布

若$X\sim N(0,1),Y\sim\mathcal{X}_n^2$，且$X,Y$相互独立

$t=X/\sqrt{Y/n}$的密度函数为

$$
t_n(y)=\frac{\Gamma((n+1)/2)}{\sqrt{n\pi}\Gamma(n/2)}\bigg(1+\frac{y^2}{n}\bigg)^{-\frac{n+1}{2}}
$$

这个密度函数称为“自由度为$n$的$t$分布”的密度函数，常简记为$t_n$

$n$充分大时

$$
\lim_{n\to\infty}t_n(y)=\frac{1}{\sqrt{2\pi}}e^{-t^2/2}
$$

**上$\alpha$分位点**

查表可知。

具有对称性$w_{1-\alpha}=-w_\alpha$

当$n$充分大，如$n>45$时，其分位点和标准正态分布近似。

### F分布

设$U\sim\mathcal{X}_{n_1}^2,V\sim\mathcal{X}_{n_2}^2$，且$U,V$相互独立，则称随机变量

$$
F = \frac{U/n_1}{V/n_2}
$$

服从自由度为$(n_1,n_2)$的$F$分布，概率密度为

$$
f_{n_1,n_2}(y) = n_1^{n_1/2}n_2^{n_2/2}\frac{\Gamma(\frac{n_1+n_2}{2})}{\Gamma(\frac{n_1}{2})\Gamma(\frac{n_2}{2})}y^{n_1/2-1}(n_1y+n_2)^{-(n_1+n_2)/2}
$$

**性质**

若$F\sim F(n_1,n_2)$，则$\frac{1}{F}\sim F(n_2,n_1)$

**上$\alpha$分位点**

查表可知。

性质如下：

$$
F_{1-\alpha}(n_1,n_2) = \dfrac{1}{F_\alpha(n_2,n_1)}
$$

### 八大分布

#### 单正态总体

设$X\sim N(\mu,\sigma^2)$，而$X_1,X_2,\cdots,X_n$是来自正态总体$N(\mu,\sigma^2)$的样本，$\overline{X}$和$S^2$分别是样本均值和样本方差，则有

1. 

$$
\overline{X}\sim N(\mu,\sigma^2/n)\quad\frac{\overline{X}-\mu}{\sigma/\sqrt n}\sim N(0,1)
$$

2. 

$$
\frac{\sum^n_{i=1}(X_i-\mu)^2}{\sigma^2}\sim \mathcal{X}^2_n
$$

3. 

$$
\frac{(n-1)S^2}{\sigma^2}\sim\mathcal{X}^2_{n-1}\quad \frac{\sum^n_{i=1}(X_i-\overline X)^2}{\sigma^2}\sim \mathcal{X}^2_{n-1}且\overline X与S^2相互独立
$$

4. 

$$
\frac{\overline X-\mu}{S/\sqrt n}\sim t(n-1)
$$

#### 双正态总体

设$(X_1,\cdots,X_{n_1})$和$(Y_1,\cdots,Y_{n_2})$分别是来自正态总体$N(\mu_1,\sigma_1^2)$和$N(\mu_2,\sigma_2^2)$的样本，且这两个样本相互独立，设$\overline X,\overline Y$分别是样本均值

5. 

$$
\frac{(\overline X-\overline Y)-(\mu_1-\mu_2)}{\sqrt{\sigma_1^2/n_1+\sigma_2^2/n_2}}\sim N(0,1)
$$

6. 当$\sigma_1=\sigma_2$未知时

$$
\frac{(\overline X-\overline Y)-(\mu_1-\mu_2)}{S_W\sqrt{1/n_1+1/n_2}}\sim t(n_1+n_2-2)
$$

其中

$$
S_W = \sqrt{\frac{(n_1-1)S_1^2+(n_2-1)S_2^2}{n_1+n_2-2}}
$$

7. 

$$
\frac{n_2\sigma_2^2\sum^{n_1}_{i=1}(X_i-\mu_1)^2}{n_1\sigma_1^2\sum^{n_2}_{i=1}(Y_i-\mu_2)^2}\sim F(n_1,n_2)
$$

8. 

$$
\frac{S_1^2/S_2^2}{\sigma_1^2/\sigma_2^2}\sim F(n_1-1,n_2-1)
$$

# 随机变量的数字特征

## 数学期望与中位数

### 定义

**定义1**

设随机变量$X$只取有限个可能值$a_1,\cdots,a_m$，其概率分布为$P(X=a_i)=p_i(i=1,\cdots,m)$，则$X$的数学期望，记为$E(X)$或$EX$，定义为

$$
E(X) = a_1p_1+a_2p_2+\cdots+a_mp_m
$$

如果取无穷个值$a_1,a_2,\cdots$，而概率分布为$P(X=a_i)=p_i(i=1,\cdots)$，则有其数学期望为

$$
E(X) = \sum^\infty_{i=1}a_ip_i（式1）
$$

这个期望存在，必须要求这个级数收敛，并且是绝对收敛。

**定义2**

如果

$$
\sum^\infty_{i=1}|a_i|p_i<\infty
$$

则称式1的右边的级数之和为$X$的数学期望

**定义3**

设$X$有概率密度函数$f(x)$，如果

$$
\int^\infty_{-\infty}|x|f(x)dx<\infty
$$

则称

$$
E(X) = \int^\infty_{-\infty}xf(x)dx
$$

为$X$的数学期望。

### 性质

**定理1**

若干个随机变量之和的期望等于各变量的期望之和

$$
E(X_1+X_2+\cdots+X_n) = E(X_1)+E(X_2)+\cdots+E(X_n)
$$

**定理2**

若干个独立（注意独立）随机变量之积的期望等于各变量期望之积

$$
E(X_1X_2\cdots X_n) = E(X_1)E(X_2)\cdots E(X_n)
$$

**随机变量函数的期望**

设随机变量$X$为离散型，有分布$P(X=a_i)=p_i(i=1,2,\cdots)$；或者为连续型，有概率密度函数$f(x)$，则

$$
E(g(x)) = \sum_i g(a_i)p_i（要求绝对收敛）
$$

或

$$
E(g(x)) = \int^\infty_{-\infty}g(x)f(x)dx（要求\int^\infty_{-\infty}|g(x)|f(x)dx<\infty）
$$

有一个特例是，若$c$为常数，则

$$
E(cX) = cE(X)
$$

### 多维情况

以二维为例

设$Z$是随机变量$X,Y$的函数，$Z=g(X,Y)$，$g$是连续函数，$Z$是一维随机变量

*离散情况*

设$(X,Y)$的分布律为$P(X=x_i,Y=y_j)=p_{ij}$

$$
E(Z) = E[g(X,Y)] = \sum_j\sum_i g(x_i,y_j)p_{ij}
$$

*连续情况*

设$(X,Y)$的概率密度是$f(x,y)$，则有

$$
E(Z)=E[g(X,Y)]=\int^{+\infty}_{-\infty}\int^{+\infty}_{-\infty}g(x,y)f(x,y)dxdy
$$

更高维可以同理推出。

### 条件数学期望

$$
E(Y|x) = \int^\infty_{-\infty}yf(y|x)dy
$$

类似于全概率公式，我们可以求得无条件的数学期望的一个重要公式

$$
E(Y) = \int^\infty_{-\infty}E(Y|x)f_1(x)dx
$$

其中

$$
f_1 = \int^\infty_{-\infty}f(x,y)dy
$$

如果将$g(x)=E(Y|x)$，则

$$
E(Y) = \int^\infty_{-\infty}g(x)f_1(x)dx=E(g(X))=E(E(Y|X))
$$

整理得到

$$
E(Y) = E[E(Y|X)]
$$

如果是多维的，也有

$$
E(Y) = \int^\infty_{-\infty}\cdots\int^\infty_{-\infty}E(Y|x_1,\cdots,x_n)f(x_1,\cdots,x_n)dx_1\cdots dx_n
$$

如果是离散的，有$P(X=a_i)=p_i$，则

$$
E(Y) = \sum^\infty_{i=1}p_iE(Y|a_i)
$$

### 中位数

**定义1**

设连续型随机变量$X$的分布函数为$F(x)$，则满足条件

$$
P(X\leq m)=F(m)=1/2
$$

的数$m$称为$X$或分布$F$的中位数。

而连续的时候，有

$$
P(X\leq m) = P(X < m) = P(X > m) = P(X\geq m) = 1/2   
$$

## 方差与矩

### 方差与标准差

**定义1**

设$X$为随机变量，分布为$F$，则

$$
Var(X) = E(X-EX)^2
$$

称为$X$的方差，有时会记为$DX$，其算术平方根称为标准差。

将其展开得

$$
Var(X)=E(X^2)-(EX)^2
$$

**定理1**

1. 常数的方差为$0$
2. 若$c$为常数，则$Var(X+c)=Var(X)$
3. 若$c$为常数，则$Var(cX)=c^2Var(X)$
4. $Var(X)=0$的充要条件为，$P(X=E(X))=1$

**定理2**

独立（注意独立）随机变量之和的方差等于各变量的方差之和，即

$$
Var(X_1+X_2+\cdots+X_n) = Var(X_1)+Var(X_2)+\cdots+Var(X_n)
$$

如果不要求独立，二维有

$$
Var(X+Y)=Var(X)+Var(Y)+2E\{[X-E(X)][Y-E(Y)]\}
$$



### 矩

**定义1**

设$X$为随机变量，$c$为常数，$k$为正整数，则量$E[(X-c)^k]$称为$X$关于$c$点的$k$阶矩

1. $c=0$时称为$X$的$k$阶原点矩，记作$\mu_k$
2. $c=E(X)$时称为$X$的$k$阶中心矩，记作$v_k$。

另外

1. $\mu_{kl}=E(X^kY^l)$称为$X,Y$的$k+l$阶混合原点矩，简称$k+l$混合矩
2. $v_{kl}=E[(X-E(X))^k(Y-E(Y))^l]$称为$X,Y$的$k+l$阶混合中心矩

## 协方差与相关系数

对于二维随机向量$(X,Y)$，$X,Y$本身都是一维随机变量，可以定义其均值、方差，我们记为

$$
E(X)=m_1,E(Y)=m_2,Var(X)=\sigma_1^2,Var(Y)=\sigma_2^2
$$

**定义1**

称$E[(X-m_1)(Y-m_2)]$为$X,Y$的协方差，并记为$Cov(X,Y)$

显然有$Cov(X,Y)=Cov(Y,X)$

也有

1. 对常数$c_1,c_2,c_3,c_4$，有

$$
Cov(c_1X+c_2,c_3Y+c_4)=c_1c_3Cov(X,Y)
$$

2. $Cov(X,Y)=E(XY)-m_1m_2=E(XY)-E(X)E(Y)$
3. $Cov(X,X)=Var(X)$
4. $Cov(X_1+X_2,Y)=Cov(X_1,Y)+Cov(X_2,Y)$
5. $Var(X\pm Y)=Var(X)+Var(Y)\pm 2Cov(X,Y)$

**定理1**

1. 若$X,Y$独立，则$Cov(X,Y)=0$
2. $[Cov(X,Y)]^2\leq\sigma_1^2\sigma_2^2$，等号当且仅当$X,Y$之间有严格线性关系（即存在常数$a,b$使$Y=a+bX$）时成立。

**定义2**

称$Cov(X,Y)/(\sigma_1\sigma_2)$为$X,Y$的相关系数，并记为$Corr(X,Y)$，有时也记为$\rho_{XY}$

可以将相关系数看做标准尺度下的协方差。

**定理2**

1. 若$X,Y$独立，则$Corr(X,Y)=0$，当然也要求方差大于$0$才有定义。
2. $-1\leq Corr(X,Y)\leq 1$，等号当且仅当$X,Y$之间有严格线性关系时达到。

需要注意几点：

1. 当$Corr(X,Y)=0$（或$Cov(X,Y)=0$）时，称$X,Y$不相关，通常我们只能由“独立”推出“不相关”，而不能反过来。但是对于服从二维正态分布的$X,Y$，“独立”和“不相关”是一回事。
2. 相关系数也常称为“线性相关系数”，相关系数并不是刻画$X,Y$之间一般关系的程度，而只刻画了“线性”关系的程度。上述定理的第二条提供了一个依据。
3. 如果$0< |Corr(X,Y)|< 1$，则解释为$X,Y$之间有一定程度的线性关系。
4. 线性相关的意义还可以从最小二乘法的角度去解释。

## 大数定理和中心极限定理

### 大数定理

**定义1**

设$X_1,X_2,\cdots$是随机变量序列，如果存在数列$a_1,a_2,\cdots$使得对任意的$\varepsilon>0$，有

$$
\lim_{n\to\infty}P\bigg(\bigg|\frac{1}{n}\sum^n_{i=1}X_i-a_n\bigg|\geq\varepsilon\bigg)=0
$$

则称随机变量序列$\{X_i\}$服从大数定律。

**定义2**

设$X_1,X_2,\cdots$是随机变量序列，$X$是随机变量，若对任意的$\varepsilon>0$，有

$$
\lim_{n\to\infty}P(|X_n-X|\geq \varepsilon)=0
$$

此时我们称$\{X_i\}$依概率收敛到$X$，记为

$$
X_n\stackrel{P}\rightarrow X,n\to\infty
$$

**马尔科夫不等式**

若$Y$为只取非负值的随机变量，则对任给常数$\varepsilon>0$，有

$$
P(Y\geq \varepsilon)\leq E(Y)/\varepsilon
$$

**切比雪夫不等式**

若$Var(Y)$存在，则

$$
P(|Y-EY|\geq\varepsilon)\leq Var(Y)/\varepsilon^2
$$

可以推知

$$
P(|Y-EY|<\varepsilon)\geq 1-Var(Y)/\varepsilon^2
$$

**切比雪夫大数定理**

设$X_1,X_2,\cdots,X_n,\cdots$是相互独立的随机变量，具有相同的期望和方差，记它们的均值都为$a$。又设它们的方差存在并记为$\sigma^2$。则对任意给定的$\varepsilon>0$，有

$$
\lim_{n\to \infty}P(|\bar X_n-a|\geq \varepsilon) = 0
$$

*更一般的形式*

设$X_1,X_2,\cdots,X_n,\cdots$是相互独立的随机变量序列，具有相同的期望$\mu$，如果存在常数$C>0$，使得$D(X_k)\leq C$，则对于任意$\varepsilon>0$，有

$$
\lim_{n\to\infty}P\bigg(\bigg|\frac{1}{n}\sum^n_{k=1}X_k-\mu\bigg|\geq\varepsilon\bigg)=0
$$

**马尔科夫大数定律**

设$X_1,X_2,\cdots,X_n,\cdots$是随机变量序列，且

$$
\lim_{n\to\infty}\frac{1}{n^2}D\bigg[\sum^n_{k=1}X_k\bigg]=0
$$

则对于任意$\varepsilon>0$，有

$$
\lim_{n\to\infty}P\bigg(\bigg|\frac{1}{n}\sum^n_{k=1}X_k-\frac{1}{n}\sum^n_{k=1}E(X_k)\bigg|\geq\varepsilon\bigg)=0
$$

**辛钦大数定律**

设$X_1,X_2,\cdots,X_n,\cdots$是相互独立且同分布的随机变量序列，具有有限的数学期望，记为$\mu$。则对于任意$\varepsilon>0$，有

$$
\lim_{n\to\infty}P\bigg(\bigg|\frac{1}{n}\sum^n_{k=1}X_k-\mu\bigg|\geq\varepsilon\bigg)=0
$$

**伯努利大数定律**

设$n_A$是$n$次独立重复实验（$n$重伯努利实验）事件$A$发生的次数，$p$是事件$A$在每次试验中发生的概率，即$P(A)=p$，则对任意的$\varepsilon>0$，有

$$
\lim_{n\to\infty} P\bigg(\bigg|\frac{n_A}{n}-p\bigg|\geq\varepsilon\bigg)=0
$$

### 中心极限定理

**定理1**

设$X_1,X_2,\cdots,X_n,\cdots$为独立同分布的随机变量，$E(X_i)=\mu,Var(X_i)=\sigma^2(0<\sigma^2<\infty)$。则对任何实数$x$，有

$$
\lim_{n\to\infty}P\bigg(\frac{1}{\sqrt{n} \sigma}(X_1+\cdots+X_n-n\mu)\leq x\bigg)=\varPhi(x)
$$

这里，$\varPhi(x)$是标准正态分布$N(0,1)$的分布函数。

也就是说，当$n$充分大时，有

$$
\sum_{k=1}^n X_k\sim N(n\mu,n\sigma^2),\quad \frac{\sum_{k=1}^nX_k-n\mu}{\sqrt{n}\sigma}\sim N(0,1)
$$

$$
P\bigg(\sum^n_{k=1}X_k\leq x\bigg)\approx\Phi\bigg(\frac{x-n\mu}{\sqrt n\sigma}\bigg),\quad P\bigg(a<\sum^n_{k=1}X_k\leq b\bigg)\approx\Phi\bigg(\frac{b-n\mu}{\sqrt n\sigma}\bigg)-\Phi\bigg(\frac{a-n\mu}{\sqrt n\sigma}\bigg)
$$

$$
\bar X\approx N(\mu,\sigma^2/n),\quad \frac{\bar X-\mu}{\sigma/\sqrt{n}}\approx N(0,1)
$$

**李雅普诺夫中心极限定理**

设$X_1,X_2,\cdots,X_n,\cdots$相互独立，其具有如下数学期望和方差：$E(X_k)=\mu_k,D(X_k)=\sigma_k^2>0$，记$B_n^2=\sum^n_{k=1}\sigma^2_k$

若存在正数$\delta$使得当$n\to\infty$时

$$
\frac{1}{B^{2+\delta}}\sum^n_{k=1}E\bigg[|X_i-\mu_i|^{2+\delta}\bigg]\to 0
$$

则

$$
\lim_{n\to\infty}P\bigg(\frac{\sum^n_{k=1}X_k-\sum^n_{k=1}\mu_k}{B_n}\leq x\bigg)=\Phi(x)
$$

**棣莫弗-拉普拉斯中心极限定理**

设$n_A$是$n$次独立重复实验（$n$重伯努利实验）事件$A$发生的次数，$p$是事件$A$在每次试验中发生的概率，即$P(A)=p$，

$$
\lim_{n\to\infty}P\bigg(\frac{n_A-np}{\sqrt{np(1-p)}}\leq x\bigg)=\Phi(x)
$$

即，若$X\sim B(n,p)$，$n$充分大时

$$
X\sim N(np,np(1-p))
$$

$$
P(X\leq x)\approx\Phi\bigg(\frac{x-np}{\sqrt{np(1-p)}}\bigg),\quad P(a < X\leq b)\approx\Phi\bigg(\frac{b-np}{\sqrt{np(1-p)}}\bigg)-\Phi\bigg(\frac{a-np}{\sqrt{np(1-p)}}\bigg)
$$

# 参数估计

## 数理统计学的基本概念

### 总体

总体是指与所研究的问题有关的对象（个体）的全体所构成的集合。

赋有一定概率分布的总体就称为统计总体。

在数理统计学中，“总体”这个基本概念的要旨——总体就是一个概率分布，当总体分布为指数分布时，称为指数分布总体；当总体分布为正态分布时，称为正态分布总体，或简称正态总体。

总体分布是一个概率分布族。这个分布族包含一个参数时，称为单参数分布族，例如指数分布。包含两个参数时，称为两参数分布族，例如正态分布。如果总体分布不能通过若干个未知参数表达出来，这种情况称为非参数总体。

### 样本

样本是按一定的规定从总体中抽出的一部分个体，所谓“按一定的规定”，就是指总体中的每一个个体有同等的被抽出的机会。

样本表现为若干个数据$X_1,\cdots,X_n,n$称为“样本大小”或“样本容量”、“样本量”，样本$X_1,\cdots,X_n$中的每一个$X_i$也称为样本。有时$X_1,\cdots,X_n$称为一组样本，而$X_i$称为其中的第$i$个样本。

### 统计量

完全由样本所决定的量叫做统计量。统计量只以来于样本，而不能依赖于任何其他未知的量。

### 简单随机样本

设$X$是具有分布函数$F$的随机变量，若$X_1,\cdots,X_n$是相互独立，且与$X$具有相同分布函数$F$的随机变量，则称$X_1,\cdots,X_n$是一个来自总体$X$的容量为$n$的简单随机样本，简称样本。

一个样本$X_1,\cdots,X_n$的观察值$x_1,\cdots,x_n$，称为样本值。

联合分布函数为

$$
F^*(x_1,x_2,\cdots,x_n) = \prod^n_{i=1}F(x_i)
$$

如果是离散型，有

$$
p^*(X_1=x_1,X_2=x_2,\dots,X_n=x_n)=\prod^n_{i=1}p(x_i)
$$

如果是连续型，有

$$
f^*(x_1,x_2,\cdots,x_n) = \prod^n_{i=1}f(x_i)
$$

### 经验分布函数

设$X_1,\cdots,X_n$是来自总体$X$的一个样本，$x_1,\cdots,x_n$，是样本$X_1,\cdots,X_n$的一组样本值，将其从小到大排列，并且重新编号为$x_1,\cdots,x_n$，则称函数 

$$
F_n(x)=\frac{x_1,\cdots,x_n中小于等于x的样本值的个数}{n} = \left\{\begin{matrix}
0,&x < x_1 \\
k/n,&x_k < x_{k+1} \\
1,&x\geq x_n
\end{matrix}\right.
$$

为总体$X$的经验分布函数。

### 常用统计量

**样本均值**

$$
\overline{X} = \frac{1}{n}\sum^n_{i=1}X_i
$$

其样本值为

$$
\overline{x} = \frac{1}{n}\sum^n_{i=1}x_i
$$

**样本方差**

$$
S^2 = \frac{1}{n-1}\sum^n_{i=1}(X_i-\overline{X})^2 = \frac{1}{n-1}\bigg(\sum^n_{i=1}X_i^2-n\overline{X}^2\bigg)
$$

其样本值为

$$
s^2 = \frac{1}{n-1}\sum^n_{i=1}(x_i-\overline{x})^2 = \frac{1}{n-1}\bigg(\sum^n_{i=1}x_i^2-n\overline{x}^2\bigg)
$$

**样本标准差**

$$
S = \sqrt{\frac{1}{n-1}\sum^n_{i=1}(X_i-\overline{X})^2}
$$

其样本值为

$$
s = \sqrt{\frac{1}{n-1}\sum^n_{i=1}(x_i-\overline{x})^2}
$$

**样本$k$阶原点矩**

$$
a_k = \frac{1}{n}\sum^n_{i=1}X_i^k
$$

样本值略。

**样本$k$阶中心矩**

$$
m_k = \frac{1}{n}\sum^n_{i=1}(X_i-\overline{X})^k
$$

样本值略

**定理1**

设总体$X$均值为$\mu$，方差为$\sigma^2$（无论何种分布），$X_1,X_2,\cdots,X_n$是来自总体$X$的一个样本，$\overline{X}$和$S^2$分别是样本均值和样本方差，则有

$$
E(\overline{X})=\mu,D(\overline{X})=\sigma^2/n,E(S^2) = \sigma^2
$$

## 矩估计、极大似然估计和贝叶斯估计

### 参数的点估计问题

设总体的分布为$f(x;\theta_1,\cdots,\theta_k)$，其形式已知（我们这里不区分连续和离散，具体情况具体处理就行），而有$k$个未知参数$\theta_1,\cdots,\theta_k$。例如对正态总体$N(\mu,\sigma^2)$，有$\theta_1=\mu,\theta_2=\sigma^2$

参数估计一般的方法就是使用总体中抽出的样本$X_1,\cdots,X_n$，去对参数$\theta_1,\cdots,\theta_k$的未知值做出估计。

为了估计$\theta_1$，我们需要构造出适当的统计量$\hat \theta_1=\hat \theta_1(X_1,\cdots,X_n)$，每当有了样本$X_1,\cdots,X_n$，就代入函数$\hat \theta_1(X_1,\cdots,X_n)$中算出一个值，用来作为$\theta_1$的估计值。

为这样的特定目的而构造的统计量$\hat\theta_1$叫做$\theta_1$的估计量。由于未知参数$\theta_1$是数轴上的一个点，用$\hat\theta_1$去估计$\theta_1$，等于用一个点去估计另一个点，所以这样的估计叫做点估计。

### 矩估计法

设总体分布为$f(x;\theta_1,\cdots,\theta_k)$，则它的矩（原点矩和中心矩都可以，此处以原点矩为例）

$$
\alpha_m = \int^\infty_{-\infty}x^mf(x;\theta_1,\cdots,\theta_k)dx
$$

或

$$
\sum_i x^m_if(x_i;\theta_1,\cdots,\theta_k)
$$

依赖于$\theta_1,\cdots,\theta_k$。另一方面，至少样本大小$n$较大时，$\alpha_m$又应接近于样本的原点矩$a_m$；于是

$$
\alpha_m = \alpha_m(\theta_1,\cdots,\theta_k)\approx a_m = \sum^n_{i=1}X_i^m/n
$$

取$m=1,\cdots,k$，并将上面的近似式改为等式，就得到一个方程组

$$
\alpha_m(\theta_1,\cdots,\theta_k)= a_m\quad (m=1,\cdots,k)
$$

解此方程组，得到根$\hat\theta_i=\hat\theta_i(X_1,\cdots,X_n)(i=1,\cdots,k)$，就以$\hat\theta_i$作为$\theta_i$的估计。如果要估计的是$\theta_1,\cdots,\theta_k$的某函数$\theta_1,\cdots,\theta_k$，则用$\hat g = \hat g(X_1,\cdots,X_k) = g(\hat\theta_1,\cdots,\hat\theta_k)$去估计它。这样定出的估计量叫做矩估计。

**正态总体的估计**

例如，设$X_1,\cdots,X_n$是从正态总体$N(\mu,\sigma^2)$中抽出的样本，要估计$\mu$和$\sigma^2$。可以用样本的一阶原点矩即样本均值$\overline{X}$去估计$\mu$，而用样本的二阶中心距$m_2$去估计$\sigma^2$。

这确实是可以的，但是我们更常用的是使用样本方差$S^2$而不是使用$m_2$，即做出了一定修正，理由之后介绍。

估计标准差也可以用$\sqrt{m_2}$，但更常用$S$。

有时要估计$\sigma/\mu$，称为总体的变异系数，其是以均值为单位去恒量的总体的标准差。可以用$\sqrt{m_2}/\overline X$，一般用$S/\overline X$

**指数分布总体的估计**

又如指数分布的参数$\lambda$，因为$1/\lambda$是总体分布的均值，所以我们可以用$\overline X$去估计。

又因为其总体方差为$1/\lambda^2$，所以$1/\lambda$也可以用$\sqrt{m_2}$或$S$去估计，具体哪种好之后回去讨论。但通常我们能用低阶矩就不用高阶矩。

### 极大似然估计法

设总体分布为$f(x;\theta_1,\cdots,\theta_k)$，$X_1,\cdots,X_n$为自这个总体中抽出的样本，则样本$(X_1,\cdots,X_n)$的分布（即其概率密度函数或概率密度）为

$$
f(x_1;\theta_1,\cdots,\theta_k)f(x_2;\theta_1,\cdots,\theta_k)\cdots f(x_n;\theta_1,\cdots,\theta_k)
$$

记为$L(x_1,\cdots,x_n;\theta_1,\cdots,\theta_k)$

当把$\theta_1,\cdots,\theta_k$而看作$x_1,\cdots,x_n$的函数时，$L$是一个概率密度函数或概率函数。$L(Y_1,\cdots,Y_n;\theta_1,\cdots,\theta_k)>L(X_1,\cdots,X_n;\theta_1,\cdots,\theta_k)$意味着在观察时出现$Y_1,\cdots,Y_n$这个点的概率比出现$X_1,\cdots,X_n$这个点的可能性大。

而反过来，$L(X_1,\cdots,X_n;\theta_1',\cdots,\theta_k')>L(X_1,\cdots,X_n;\theta_1'',\cdots,\theta_k'')$，则被估计的参数$\theta_1,\cdots,\theta_k$是$\theta_1',\cdots,\theta_k'$的可能性比它是$\theta_1'',\cdots,\theta_k''$的可能性大。此时$L$称作似然函数。

所以我们的极大似然估计法就是用似然程度最大的那个点$(\theta_1^*,\cdots,\theta_k^*)$，即满足条件

$$
L(X_1,\cdots,X_n;\theta^*_1,\cdots,\theta^*_k)=\underset{\theta_1,\cdots,\theta_k}{\max}L(X_1,\cdots,X_n;\theta_1,\cdots,\theta_k)
$$

的$(\theta_1^*,\cdots,\theta_k^*)$作为$(\theta_1,\cdots,\theta_k)$的估计值。这个估计值就叫做极大似然估计，估计函数$g(\theta_1,\cdots,\theta_n)$就用$g(\theta_1^*,\cdots,\theta_n^*)$

因为

$$
\ln L = \sum^n_{i=1}\ln f(X_i;\theta_1,\cdots,\theta_k)
$$

为使$L$最大，则建立似然方程组如下

$$
\dfrac{\partial\ln L}{\partial\theta_i} = 0\quad(i=1,\cdots,k)
$$

如果有唯一解，且是极大值点，则它必是使$L$达到最大的点。但有时候解不唯一，并且判断哪个使$L$达到极大也不容易。

有时$f$不连续，或者不可导，这时就不能列方程，要回到最原始的定义。

**正态总体**

作为例子，正态总体用极大似然估计是可以的，它连续，并且可导，计算后得到的是$\mu^*=\overline X,\sigma^{*2}=m_2$

**指数分布总体**

估计得$\lambda=1/\overline X$

上面的两个例子，其和矩估计恰好一致，但这只是一种巧合，更多时候是不一致的。但这两种估计方法结果一致，说明这些估计是良好的。

另外我们也可以应用样本中位数$\hat m$去估计正态总体的$\mu$，这个统计量可以在矩估计不能使用（主要体现为矩为无限大等场景）和极大似然法求根不容易等场景。

### 贝叶斯法

在矩估计和极大似然估计时，在我们心目中，未知参数$\theta$就简单地是一个未知数，在抽取样本之前，我们对$\theta$没有任何了解，所有的信息全来自样本。

贝叶斯学派则不然，它的出发点是：在进行抽样之前，我们已对$\theta$有一定的知识，叫做先验知识，即试验之前的知识，也叫做验前知识。

贝叶斯学派进一步要求：这种先验知识必须用$\theta$的某种概率分布表达出来，这个概率分布就叫做$\theta$的“先验分布”或“验前分布”。这个分布总结了我们在试验之前对未知参数$\theta$的知识。

比如我们可以使用工厂以前制造的产品废品率来作为先验知识。以$h(\theta)$代表先验密度。但如果这个工厂是新开的，无法得到$h(\theta)$，则我们必须设法定出一个$h(\theta)$，甚至是自己的主观估计也可以。

当我们确定先验密度后，设总体有概率密度$f(X,\theta)$，从这个总体中抽样本$X_1,\cdots,X_n$，则这组样本的密度为$f(X_1,\theta)\cdots f(X_n,\theta)$。它可视为在给定$\theta$时$(X_1,\cdots,X_n)$的密度，$(\theta,X_1,\cdots,X_n)$的联合密度为

$$
h(\theta)f(X_1,\theta)\cdots f(X_n,\theta)
$$

由此，算出$(X_1,\cdots,X_n)$的边缘密度为

$$
p(X_1,\cdots,X_n) = \int h(\theta)f(X_1,\theta)\cdots f(X_n,\theta)d\theta
$$

积分的范围依$\theta$的具体范围而定。然后有

$$
h(\theta|X_1,\cdots,X_n) = h(\theta)f(X_1,\theta)\cdots f(X_n,\theta)/p(X_1,\cdots,X_n)
$$

按照贝叶斯学派的观点，这个条件密度代表了我们现在取得样本$X_1,\cdots,X_n$后对$\theta$的知识，它综合了先验知识和样本的信息。这个式子称为后验（验后）密度。

贝叶斯学派也指出，在得出后验分布后，对参数$\theta$的任何统计推断都只能基于这个后验分布。

## 点估计的优良性准则

### 估计量的无偏性

设某统计总体的分布包含未知参数$\theta_1,\cdots,\theta_k$，$X_1,\cdots,X_n$是从该总体中抽出的样本，要估计$g(\theta_1,\cdots,\theta_k)$，$g$为一已知函数，设$\hat g(X_1,\cdots,X_n)$是一个估计量。如果对任何可能的$\theta_1,\cdots,\theta_k$都有

$$
E_{\theta_1,\cdots,\theta_k}[\hat g(X_1,\cdots,X_n)]=g(\theta_1,\cdots,\theta_k)
$$

则称$\hat g$是$g(\theta_1,\cdots,\theta_k)$的一个无偏估计量。

估计量的无偏性有两个含义：

1. 没有系统性的误差
2. 根据大数定理，若估计量有无偏性，则在大量次数使用取平均时，能以接近于$100\%$的把握无限逼近被估计的量。如果没有无偏性，那么估计值会和真值保持一定距离，这个距离就是系统误差。

可以证明

**例1**

设$X_1,\cdots,X_n$是从总体中抽出的样本，则样本均值$\overline X$是总体分布均值$\theta$的无偏估计。

**例2**

样本方差$S^2$是总体分布方差$\sigma^2$的无偏估计。（而不是二阶中心矩）

### 最小方差无偏估计

一个参数往往有不止一个无偏估计，我们要从中调出一个最优的。

#### 均方误差

设$X_1,\cdots,X_n$是从某一带参数$\theta$的总体中抽出的样本，要估计$\theta$。若我们采用估计量$\hat\theta = \hat\theta(X_1,\cdots,X_n)$，则其误差为$\hat\theta(X_1,\cdots,X_n)-\theta$。这个误差随样本的具体值而定，也是随机的，无法作为优良性的指标。我们取

$$
M_{\hat\theta}(\theta)=E_\theta[\hat\theta(X_1,\cdots,X_n)-\theta]^2
$$

作为$\hat\theta$的误差大小从整体角度的一个衡量。这个量越小，就表示$\hat\theta$的误差平均来讲比较小，因而也就越优。$M_{\hat\theta}$就称为估计量$\theta$的“均方误差”

#### 最小方差无偏估计

从前面的讨论看到：若局限于无偏估计的范围，且采用均方误差的准则，则两个无偏估计$\hat\theta_1$和$\hat\theta_2$的比较归结为其方差的比较：方差小者为优。

最小方差无偏估计简记为MVU估计。

#### 克拉美-劳不等式

TODO

### 估计量的相合性与渐近正态性

#### 相合性

设总体分布依赖于参数$\theta_1,\cdots,\theta_k,g(\theta_1,\cdots,\theta_k)$是$\theta_1,\cdots,\theta_k$的一个给定函数，设$X_1,\cdots,X_n$
为自该总体中抽出的样本，$T(X_1,\cdots,X_n)$是$g(\theta_1,\cdots,\theta_k)$的一个估计量，如果对任给$\varepsilon>0$有

$$
\lim_{n\to\infty}P_{\theta_1,\cdots,\theta_k}(|T(X_1,\cdots,X_n)-g(\theta_1,\cdots,\theta_k)|\geq\varepsilon) = 0
$$

而且这对$(\theta_1,\cdots,\theta_k)$一切可能取的值都成立，则称$T(X_1,\cdots,X_n)$是$g(\theta_1,\cdots,\theta_k)$的一个相合估计。

#### 渐近正态性

正如在中心极限定理中所显示的，当$n$很大时，和的分布渐近于正态分布。理论上可以证明，这不只是和所独有的，许多形状复杂的统计量，当样本大小$n\to\infty$时，其分布都渐近于正态分布。这称为统计量的“渐近正态性”。

## 区间估计

### 基本概念

点估计是用一个点去估计未知参数。区间估计就是用一个区间去估计未知参数，即未知参数的值也会估计在一个区间之内。

设$X_1,\cdots,X_n$是从该总体中抽出的样本。所谓$\theta$的区间估计，就是以满足条件$\hat\theta_1(X_1,\cdots,X_n)\leq\hat\theta_2(X_1,\cdots,X_n)$的两个统计量$\hat\theta_1,\hat\theta_2$为端点的区间$[\hat\theta_1,\hat\theta_2]$。一旦有了样本$X_1,\cdots,X_n$，就把$\theta$估计在区间$[\hat\theta_1(X_1,\cdots,X_n),\hat\theta_2(X_1,\cdots,X_n)]$之内。有两个要求

1. $\theta$要以很大的可能性落在$[\hat\theta_1,\hat\theta_2]$之内，也就是说

$$
P_\theta(\hat\theta_1\leq\theta\leq\hat\theta_2)
$$

要尽可能大。

2. 估计的精密度要尽可能高。比方说，要求区间的长度$\hat\theta_2-\hat\theta_1$尽可能小，或某种能体现这个要求的其他准则。

**置信系数、置信区间、置信水平**

给定一个很小的数$\alpha>0$。如果对参数$\theta$的任何值，$P_\theta(\hat\theta_1\leq\theta\leq\hat\theta_2)$都等于$1-\alpha$，则层区间估计$[\hat\theta_1,\hat\theta_2]$的置信系数为$1-\alpha$，而这个区间叫做置信区间。

有时，我们无法证明恰好等于$1-\alpha$，但我们能证明它大于等于$1-\alpha$，那我们称$1-\alpha$是$[\hat\theta_1,\hat\theta_2]$的置信水平。当然，如果$0.8$是，那么$0.7,0.6$等等也都是，置信系数就是置信水平中的最大者。

### 枢轴变量法

1. 找一个与要估计的参数$g(\theta)$有关的统计量$T$，一般是其一个良好的点估计
2. 设法找出$T$和$g(\theta)$的某一函数$S(T,g(\theta))$，其分布$F$要与$\theta$无关，$S$称为枢轴变量。
3. 对任何常数$a< b$，不等式$a\leq S(T,g(\theta))\leq b$要能改写为等价的形式$A\leq g(\theta)\leq B$，$A,B$只与$T,a,b$有关，而与$\theta$无关
4. 取分布$F$的上$\alpha/2$分位点$w_{\alpha/2}$和上$1-\alpha/2$分位点$w_{1-\alpha/2}$，则有$F(w_{\alpha/2})-F(w_{1-\alpha/2})=1-\alpha$，因此

$$
P(w_{1-\alpha/2}\leq S(T,g(\theta))\leq w_{\alpha/2})=1-\alpha
$$

然后根据第三条改写为$A\leq g(\theta)\leq B$，则$[A,B]$就是$g(\theta)$的一个置信系数为$1-\alpha$的区间估计。

上$\beta$分位点的含义是，对于任何分布$F$，满足条件$F(v_\beta)=1-\beta$的点$v_\beta$就是分布函数$F$的上$\beta$分位点。

下面给出一些常见的枢轴量

**单正态总体**

*检验均值$\mu$*

若$\sigma^2$已知，则枢轴量

$$
U = \dfrac{\overline X-\mu}{\sigma/\sqrt n}\sim N(0,1)
$$

若$\sigma^2$未知，则枢轴量

$$
t = \dfrac{\overline X-\mu}{S/\sqrt n}\sim t(n-1)
$$

*检验方差$\sigma^2$*

若$\mu$已知，则枢轴量

$$
\mathcal{X}^2 = \dfrac{\displaystyle\sum^n_{i=1}(X_i-\mu)^2}{\sigma^2}\sim \mathcal{X}^2_n
$$

若$\mu$未知，则枢轴量

$$
\mathcal{X}^2 = \dfrac{(n-1)S^2}{\sigma^2}\sim \mathcal{X}^2_{n-1}
$$

**两个相互独立的正态总体**

*检验均值差$\mu_1-\mu_2$*

若$\sigma^2_1,\sigma^2_2$已知，则枢轴量

$$
U = \dfrac{(\overline X-\overline Y)-(\mu_1-\mu_2)}{\sqrt{\sigma^2_1/n_1+\sigma^2_2/n_2}}\sim N(0,1)
$$

若$\sigma^2_1=\sigma^2_2=\sigma^2$未知，则枢轴量

$$
t = \dfrac{(\overline X-\overline Y)-(\mu_1-\mu_2)}{S_W\sqrt{1/n_1+1/n_2}}\sim t(n_1+n_2-2)
$$

*检验方差比$\sigma_1^2/\sigma_2^2$*

若$\mu_1,\mu_2$已知，则枢轴量

$$
F = \dfrac{n_2\sigma_2^2\sum^{n_1}_{i=1}(X_i-\mu_1)^2}{n_1\sigma_1^2\sum^{n_2}_{i=1}(Y_i-\mu_2)^2}\sim F(n_1,n_2)
$$

若$\mu_1,\mu_2$未知，则枢轴量

$$
F = \dfrac{\sigma^2_2S_1^2}{\sigma_1^2S_2^2}\sim F(n_1-1,n_2-1)
$$

**枢轴量的使用方法**

我们最终要从枢轴量中得到置信区间，例如检验均值$\mu$

若$\sigma^2$已知，则枢轴量

$$
U = \dfrac{\overline X-\mu}{\sigma/\sqrt n}\sim N(0,1)
$$

也就是我们要

$$
P(w_{1-\alpha/2}\leq U\leq w_{\alpha/2}) = P(-w_{\alpha/2}\leq U\leq w_{\alpha/2})=1-\alpha
$$

我们就将$\mu$的表达式写出来为$[\overline X - \dfrac{\sigma}{\sqrt n}w_{\alpha/2},\overline X + \dfrac{\sigma}{\sqrt n}w_{\alpha/2}]$

其他的使用办法也是类似的。

### 大样本法

TODO

### 置信界

设$X_1,\cdots,X_n$是从某一总体中抽出的样本，总体分布包含未知参数$\theta$，$\overline{\theta}=\overline{\theta}(X_1,\cdots,X_n)$和$\underline{\theta}=\underline{\theta}(X_1,\cdots,X_n)$都是统计量，则

1. 若对$\theta$的一切可取的值，有

$$
P_\theta(\overline{\theta}(X_1,\cdots,X_n)\geq\theta)=1-\alpha
$$

则称$\overline\theta$是$\theta$的一个置信系数为$1-\alpha$的置信上界。

2. 若对$\theta$的一切可取的值，有

$$
P_\theta(\underline{\theta}(X_1,\cdots,X_n)\leq\theta)=1-\alpha
$$

则称$\underline\theta$是$\theta$的一个置信系数为$1-\alpha$的置信下界。

单侧置信区间使用枢轴量的方法类似，只不过要将$P$函数代成单侧的情况的。对于置信上界，置信区间的下限为$-\infty$，对于置信下界，置信区间的上限为$+\infty$

### 贝叶斯法

TODO

# 假设检验

## 基本概念

在显著性水平$\alpha$之下，检验假设$H_0$和$H_1$。$H_0$称为原假设或零假设，$H_1$称为备择假设，备择假设是指与原假设相矛盾的假设。

**双边假设检验**

$$
H_0 : \mu=\mu_0\quad H_1 : \mu\neq\mu_0
$$

**单边假设检验**

*右边假设*

$$
H_0 : \mu\leq\mu_0\quad H_1 : \mu>\mu_0
$$

*左边假设*

$$
H_0 : \mu\geq\mu_0\quad H_1 : \mu<\mu_0
$$

仅涉及总体分布未知参数的假设称为参数假设。而对总体分布类型或分布的某些特征提出的假设称为非参数假设。

能够对原假设$H_0$是真或不真做出回答的统计量称为假设检验统计量。

使得原假设$H_0$为真的假设检验统计量取值范围称为假设检验的接受域；拒绝原假设$H_0$的假设检验统计量取值范围称为拒绝域；拒绝域的边界点称为临界点。

由于样本的随机性，进行判断时，可能会发生错误，发生错误也是一个随机事件

**第I类错误（弃真）：$H_0$为真但拒绝$H_0$**

$$
P(拒绝H_0/H_0为真时) = \alpha
$$

**第II类错误（存伪）：$H_0$不真但接受$H_0$**

$$
P(接受H_0/H_0不真时) = \beta
$$

给定样本容量，如果减小第I类错误的概率，则第II类错误的概率往往增大。如果要使两类错误的概率都减小，需要增大样本容量。

我们一般控制第I类错误的概率使其不大于$\alpha$，通常取$\alpha$为$0.1,0.05,0.01,0.005$

只控制第I类错误的概率，而不考虑第II类错误的概率的假设检验称为显著性检验，称$\alpha$为显著性水平。

**正态总体假设检验的接受域**

设总体$X\sim N(\mu,\sigma^2),\mu$未知，$\sigma^2$已知，设$X_1,X_2,\cdots,X_n$是来自总体的样本，给定显著性水平为$\alpha$

*双边假设*

接受域为

$$
|u| = \bigg|\dfrac{\bar x-\mu_0}{\sigma/\sqrt n}\bigg| < z_{\alpha/2}
$$

*左边假设*

接受域为

$$
u = \dfrac{\bar x-\mu_0}{\sigma/\sqrt n}>-z_\alpha
$$

*右边假设*

接受域为

$$
u = \dfrac{\bar x-\mu_0}{\sigma/\sqrt n} < z_\alpha
$$

**求未知参数假设检验的步骤**

1. 根据实际问题，提出原假设$H_0$和备择假设$H_1$
2. 确定检验统计量及其在$H_0$为真时的分布
3. 根据显著性水平$\alpha$和样本容量$n$，按照$P(当H_0为真时拒绝H_0)=\alpha$求出临界点，确定接受域或拒绝域
4. 计算检验统计量的样本观测值
5. 根据样本观测值做出决策，是接受原假设$H_0$还是拒绝$H_0$

双边假设的两个临界点：上$\alpha/2$分位点、上$1-\alpha/2$分位点

左边假设的一个临界点：上$1-\alpha$分位点

右边假设的一个临界点：上$\alpha$分位点

## 单正态总体和双正态总体参数的假设检验

可以直接用之前提到枢轴量，以及其分位点来计算，具体形式和“正态总体假设检验的接受域”一小节类似，不再介绍。
