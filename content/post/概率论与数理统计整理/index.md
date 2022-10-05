---
title: "概率论与数理统计整理"
date: 2022-08-30T10:41:24+08:00
draft: false
tags: [数学,大学]
description: 概率论与数理统计的学习笔记
categories: 数学
image: "cover.jpg"
mathjax: true
markup: pandoc
---

## 基本概念

### 随机试验E

针对随机现象的观察、记录、试验（广义）。

特点：

1. 可以在相同的条件下重复进行
2. 每次试验的可能结果不止一个，并且能事先明确试验的所有可能结果
3. 但一次试验前不能确定哪个结果会出现

### 样本空间$\Omega$

随机试验$E$的所有结果构成的集合为$E$的样本空间$\Omega$，记为$\Omega\{\omega\}$，每个结果$\omega$是$\Omega$中的一个元素，称为样本点。

### 频率

在相同的条件下，进行了$n$次试验，其中事件$A$发生的次数$n_A$称为事件$A$发生的频数，比值$n_A/n$称为事件$A$发生的频率，记为$f_n(A)$

### 概率

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

### 两个公式

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
P(X\leq x) = F(x)\quad(-\infty<x<\infty)
$$

称为$X$的分布函数。这里不限制$X$是离散型的。

$$
F(x) = P(X\leq x) = \sum_{\{i|a_i\leq x\}}p_i
$$

分布函数具有如下性质

1. $F(x)$是单调非降的：$x_1\leq x_2$时，有$F(x_1)\leq F(x_2)$
2. $x\to \infty$时，$F(x)\to 1$。$x\to -\infty$时，$F(x)\to 0$

## 二项分布

设某个事件$A$在一次试验中发生的概率为$p$，重复试验$n$次，以$X$记$A$在$n$次试验中发生的次数，则

$$
p_i=b(i;n,p)=\binom{n}{i}p^i(1-p)^{n-i}\quad (n=0,1,\cdots,n)
$$

服从有两个条件

1. 各次试验的条件是稳定的
2. 各次试验的独立性

如果抽检后放回，则每次抽出废品的概率是相等的。如果抽检后不放回，则废品率发生了变化，试验条件不稳定，不符合二项分布。但是如果总数远大于抽出的数量，则不放回也几乎不影响，可以近似地作为二项分布。

## 泊松分布

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

## 超几何分布

以$X$记从含$M$个废品的$N$个产品中随机抽出$n$个里面所含有的废品数，$X$的分布为

$$
P(X=m)=\binom{M}{n}\binom{N-M}{n-m}/\binom{N}{n}
$$

这是一种抽检不放回的试验，如果$N$特别大，$n$不够大时，放回与不放回差别不大。可以当作二项分布来看。或者说，若$X$服从超几何分布，则当$n$固定时，$M/N=p$固定；$N\to \infty$时，$X$近似服从二项分布$B(n,p)$。

## 负二项分布

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

## 几何分布

当负二项分布中的$r=1$时，有

$$
P(X=i) = p(1-p)^i
$$

## 连续型随机变量

与离散型的有限或可数无限不同，这是一种充满整个区间的随机变量，或者说不可数无限。

刻画连续型随机变量可以用之前提到的概率分布函数，但是更常用概率密度函数。

### 概率密度函数

设连续型随机变量$X$有概率分布函数$F(x)$，则$F(x)$的导数$f(x)=F'(x)$称为$X$的概率密度函数。

具有如下性质

1. $f(x)\geq 0$
2. $\int^\infty_{-\infty}f(x)dx=1$
3. 对于任何常数$a<b$，有

$$
P(a\leq X\leq b)=F(b)-F(a) = \int^b_af(x)dx
$$

## 正态分布

如果一个随机变量具有概率密度函数

$$
f(x)=(\sqrt{2\pi}\sigma)^{-1}e^{-\frac{(x-\mu)^2}{2\sigma^2}}\quad (-\infty<x<\infty)
$$

则称$X$为正态随机变量，并记为$X\sim N(\mu,\sigma^2)$。$\mu,\sigma^2$是常数，$\mu\in R,0<\sigma^2<\infty$，它们称为这个分布的参数。

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

可以查表代入。但是通常只有非负数的值，对于负数，有

$$
\varPhi = 1-\varPhi(-x)
$$

## 指数分布

若随机变量$X$有概率密度函数

$$
f(x)=\left\{\begin{matrix}
\lambda e^{\lambda x},\quad x>0\\
0,\quad x\leq 0
\end{matrix}\right.
$$

则$X$服从指数分布，其中$\lambda>0$是参数。

其概率分布函数为

$$
F(x)=\left\{\begin{matrix}
1-e^{-\lambda x},\quad x>0\\
0,\quad x\leq 0
\end{matrix}\right.
$$

## 威布尔分布

满足

$$
f(x)=\left\{\begin{matrix}
\lambda\alpha x^{\alpha-1} e^{\lambda x^\alpha},\quad x>0\\
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

## 均匀分布

满足

$$
f(x)=\left\{\begin{matrix}
1/(b-a),\quad a\leq x\leq b\\
0,\quad else
\end{matrix}\right.
$$

其概率分布函数是

$$
F(x)=\left\{\begin{matrix}
0,\quad\quad\quad\quad\quad\quad\quad\quad x\leq a\\
(x-a)/(b-a),\quad a<x<b\\
1,\quad\quad\quad\quad\quad\quad\quad\quad x\geq b
\end{matrix}\right.
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

**多项分布**

设$A_1,A_2,\cdots,A_n$是某一试验之下的完备事件群，即事件$A_1,\cdots,A_n$两两互斥，其和为必然事件。分别以$p_1,p_2,\cdots,p_n$记每个事件对应的概率。

现将试验独立重复$N$次，而以$X_i$记载着$N$次试验中事件$A_i$出现的次数。$X$的概率分布就叫做多项分布，有时记为$M(N;p_1,\cdots,p_n)$。其概率有

$$
P(X_1=k_1,X_2=k_2,\cdots,X_n=k_n)=\frac{N!}{k_1!k_2!\cdots k_n!}p_1^{k_1}p_2^{k_2}\cdots p_n^{k_n}
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

**二维随机向量的均匀分布**

$$
f(x_1,x_2)=
\left\{\begin{matrix}
1/[(b-a)(d-c)],\quad a\leq x_1\leq b, c\leq x_2\leq d\\
0,\quad\quad\quad\quad\quad\quad\quad\quad\quad other\quad\quad\quad\quad\quad\quad
\end{matrix}\right.
$$

**二维正态分布**

$$
f(x_1,x_2)=(2\pi\sigma_1\sigma_2\sqrt{1-\rho^2})^{-1}\exp\left[-\frac{1}{2(1-\rho^2)}\left(\frac{(x_1-a)^2}{\sigma_1^2}-\frac{2\rho(x_1-a)(x_2-b)}{\sigma_1\sigma_2}+
\\
\frac{(x_2-a)^2}{\sigma_2^2}\right)\right]
$$

常把这个分布记为$N(a,b,\sigma_1^2,\sigma_2^2,\rho)$

与一维的情况一样，也可以用概率分布函数去描述多维随机向量的概率分布，其定义为

$$
F(x_1,x_2,\cdots,x_n)=P(X_1\leq x_1, X_2\leq x_2,\cdots,X_n\leq x_n)
$$

其基本性质，以二维为例，有

1. $F(x,y)$是$x,y$的不减函数
2. $0\leq F(x,y)\leq 1,F(-\infty,-\infty) = 0,F(\infty,\infty)=1$

对于任意$x$，$F(x,-\infty)=0$，对于任意$y$，$F(-\infty,y)=0$

3. $F(x,y)$关于$x,y$均为右连续，$F(x+0,y) = F(x,y)$，$F(x,y+0) = F(x,y)$
4. 若$x_1<x_2,y_1<y_2$，则$F(x_2,y_2)+F(x_1,y_1)-F(x_2,y_1)-F(x_1,y_2)\geq 0$

另外，有

$$
f(x,y) = \frac{\partial^2F(x,y)}{\partial x\partial y}
$$

**边缘分布**

设$X=(X_1,\cdots,X_n)$为一个$n$维随机向量。$X$有一定的分布$F$，这是一个$n$味分布，因为$X$的每个分量$X_i$都是一维随机变量，故它们都有各自的分布$F_i$，这些都是一维分布，称为随机向量$X$或其分布$F$的边缘分布。边缘分布完全由原分布$F$确定。

例如

$$
P(X_1=a_{1k})=\sum_{j_2,\cdots,j_n}p(k,j_2,\cdots,j_n)
$$

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

如果连续变量$X_1,\cdots,X_n$相互独立，则对任何$a_i<b_i$，则由下式定义的$n$个事件也独立

$$
A_1=\{a_1\leq X_1\leq b_1\},\cdots,A_n=\{a_n\leq X_n\leq b_n\}
$$

反之，若对任何$a_i<b_i$，事件$A_1,\cdots,A_n$独立，则变量$X_1,\cdots,X_n$独立。

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

设$X_1,\cdots,X_n$是离散型随机变量，弱队任何常数$a_1,\cdots,a_n$都有

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
