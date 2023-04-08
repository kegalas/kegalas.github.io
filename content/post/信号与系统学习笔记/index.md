---
title: "信号与系统学习笔记"
date: 2023-04-05T20:24:29+08:00
draft: false
tags: [信号,大学,电路]
categories: 电路与信号
mathjax: true
markup: pandoc
image: "cover.jpg"
---

# 信号与系统概论

## 信号的能量与功率

对于连续时间信号$x(t)$，在$t_1\leq t\leq t_2$内的总能量为

$$
\int^{t_2}_{t_1}|x(t)|^2 dt
$$

$|\cdot|$指的是模，因为可能是复数。

在$n_1\leq n\leq n_2$内的离散时间信号$x[n]$的总能量为

$$
\sum^{n_2}_{n=n_1}|x[n]|^2
$$

功率将能量除以时间间隔即可。

能量有限信号指$E<\infty$的信号，此时$P=0$（无限长时间意义上）

功率有限信号指$0<P<\infty$的信号，此时$E=\infty$（无限长时间意义上）

时限信号为能量信号，周期信号是功率信号，非周期信号可能是其中之一，有些信号两种都不是。

## 自变量的变换

### 时移

### 时间翻转

### 时间尺度变换

### 周期信号

基波周期就是最小正周期。其对应的频率称为基波频率。

两个周期信号的周期分别为$T_1$和$T_2$，若$T_1/T_2$为有理数，则周期信号之和仍然是周期信号，其周期为$T_1$和$T_2$的最小公倍数。

离散的情况下，$sin(\beta k)$是否为周期信号？

1. 如果$2\pi/k$是整数，则周期为$N=2\pi/\beta$
2. 如果$2\pi/k$是有理数，则周期为$N=M(2\pi/\beta)$，$M$为使$N$称为正整数的最小正整数
3. 如果$2\pi/k$是无理数，则不是周期信号。

### 偶信号与奇信号

任何信号都能分解成一个偶信号和一个奇信号的和。

## 单位冲激函数与单位阶跃函数

单位冲激的符号是$\delta$，单位阶跃的符号是$\varepsilon$或$u$

在离散情况下

$$
\delta[n] = u[n]-u[n-1]
$$

是一次差分关系

而

$$
u[n] = \sum^n_{m=-\infty}\delta [m]
$$

连续情况下

$$
\delta(t) = \dfrac{du(t)}{dt}
$$

$$
u(t) = \int^t_{-\infty}\delta(\tau)d\tau
$$

## 连续时间系统和离散时间系统

### 定义

### 系统的互联

需要记住的是，并联在一起的是用加法，级联在一起的是用卷积。

## 系统的基本性质

### 有记忆与无记忆

一个系统的输出仅仅取决于该时刻的输入，则为无记忆系统。

### 可逆系统

一个系统在不同的输入下，导致不同的输出，那么就是可逆的。

### 因果性

一个系统在任何时刻的输出只取决于现在的输入和过去的输入，则称为因果系统。

因果系统的零状态响应不会出现在激励之前。

### 稳定性

一个稳定系统，若其输入是有界的，那么输出也必须是有界的。

### 时不变性

若系统的特性和行为不随时间改变，那么该系统就是时不变的。

如果在输入信号上有一个时移，而在输出信号中产生同样的时移，那么就是时不变的。

$$
T[f(t-t_0)] = y(t-t_0)
$$

有一个直观的办法可以判定一个系统是时变的。如果$f(\cdot)$前出现变系数，或者有反转、伸缩变换，则一定是时变系统。

### 线性

一个线性系统应该满足

1. $y_1(t)+y_2(t)$是对$x_1(t)+x_2(t)$的响应。
2. $ay_1(t)$是对$ax_1(t)$的响应，此处$a$为任意复常数。

可以记作

$$
T[af_1(\cdot)+bf_2(\cdot)] = aT[f_1(\cdot)]+bT[f_2(\cdot)]
$$

### 动态系统

动态系统不仅和激励$\{f(\cdot)\}$有关，还与它过去的历史状况$\{x(0)\}$有关。

*完全响应*

$$
y(\cdot) = T[\{f(\cdot)\},\{x(0)\}]
$$

*零状态响应*

$$
y_{zs}(\cdot) = T[\{f(\cdot)\},\{0\}]
$$

*零输入响应*

$$
y_{zi}(\cdot) = T[\{0\},\{x(0)\}]
$$

**当动态系统满足下列三个条件时，可以成为线性系统**

1. 可分解性

$$
y = y_{zs} + y_{zi}
$$

2. 零状态线性

$$
T[\{af_1(\cdot)+bf_2(\cdot)\},\{0\}] = aT[\{f_1(\cdot)\},\{0\}]+bT[\{f_2(\cdot)\},\{0\}]
$$

3. 零输入线性

$$
T[\{0\},\{ax_1(0)+bx_2(0)\}] = aT[\{0\},\{x_1(0)\}]+bT[\{0\},\{x_2(0)\}]
$$

# 线性时不变系统

## 卷积

离散情况

$$
a[n]*b[n] = \sum^{+\infty}_{k=-\infty} a[k]b[n-k]
$$

连续情况

$$
a(t)*b(t) = \int^{+\infty}_{-\infty}a(\tau)b(t-\tau)d\tau
$$

## 用脉冲表示信号，及其响应

以下系统指线性时不变系统（LTI）。

脉冲信号的筛选性质：

$$
x[n] = \sum^{+\infty}_{k=-\infty} x[k]\delta[n-k] = x[n]*\delta[n]
$$

$$
x(t) = \int^{+\infty}_{-\infty}x(\tau)\delta(t-\tau)d\tau = x(t)*\delta(t)
$$

一个系统对脉冲信号的响应记作$h$。一个系统的特征可以完全由脉冲响应刻画，一个系统对其他信号的响应可以用该信号和脉冲响应的卷积表示。

$$
y[n] = x[n]*h[n]
$$

$$
y(t) = x(t)*h(t)
$$

## 线性时不变系统的性质

也即线性时不变下卷积的性质

1. 交换律
2. 分配律
3. 结合律

### 无记忆的LTI 

因为输出只和当前的输入有关，所以其冲激响应为$h(t)=K\delta(t)$。

系统对于其他信号的响应就为$y(t)=Kx(t)$。离散的情况类似。

### 微分特性

$$
if\quad x(t)\to y(t)\quad then\quad x'(t)\to y'(t)
$$

### 积分特性

$$
if\quad x(t)\to y(t)\quad then\quad \int^t_{-\infty}x(t)dt\to \int^t_{-\infty}y(t)dt
$$

### 可逆性

仅当存在一个逆系统，其与原系统级联后所产生的输出等于第一个系统的输入时，这个系统才是可逆的。

设一个系统的冲激响应是$h(t)$，逆系统的冲激响应是$h_1(t)$，则

$$
h(t)*h_1(t) = \delta(t)
$$

离散的情况类似。

### 因果性

如果$h(t)=0,t<0$，那么一个LTI就是因果的，这时

$$
y(t) = \int^t_{-\infty} x(\tau)h(t-\tau)d\tau = \int^\infty_0 h(\tau)x(t-\tau)d\tau
$$

### 稳定性

### LTI的单位阶跃响应

单位阶跃响应记作$s$或$g$

$$
s[n] = u[n]*h[n] = \sum^n_{k=-\infty}h[k]
$$

$$
h[n] = s[n]-s[n-1]
$$

$$
s(t) = \int^t_{-\infty} h(\tau)d\tau
$$

$$
h(t) = \dfrac{ds(t)}{dt}
$$

## 用微分方程和差分方程描述的因果线性时不变系统

### 微分方程

微分方程的形式如下

$$
y^{(n)}(t) + a_{n-1}y^{(n-1)}(t)+\cdots+a_{1}y^{(1)}(t)+a_0y(t) = b_{m}f^{(m)}(t)+b_{m-1}f^{(m-1)}(t)+\cdots+b_{1}f^{(1)}(t)+b_{0}f(t)
$$

解的形式如下

$$
y(t) = y_h(t) + y_p(t)
$$

其中$y_h$是齐次解，$y_p$是特解。

齐次解是对应的齐次方程

$$
y^{(n)}(t) + a_{n-1}y^{(n-1)}(t)+\cdots+a_{1}y^{(1)}(t)+a_0y(t) = 0
$$

的解。

解这个方程，我们首先要求得特征根$\lambda$，由以下方程解出

$$
\lambda^n + a_{n-1}\lambda^{n-1}+\cdots+a_0=0
$$

对于每一个$\lambda_i$，

1. 如果它是单实根，则

$$
y_h = Ce^{\lambda t}
$$

2. 如果它是二重实根，则

$$
y_h = (C_1t+C_0)e^{\lambda t}
$$

3. 如果是一对共轭负根$\lambda_{1,2} = \alpha\pm j\beta$，则

$$
e^{\alpha t}[C\cos(\beta t)+D\sin(\beta t)]
$$

整个齐次解就是把这些对于每一个$\lambda_i$的齐次解加起来，再根据条件求出待定系数。

特解的常见形式为

1. 激励为$f(t)=t$时，如果所有的特征根均不等于$0$，

$$
P_1 t+P_0
$$

如果有$1$个等于$0$的特征根，

$$
t(P_1t+P_0)
$$

2. 激励为$f(t)=e^{\alpha t}$，如果$\alpha$不等于特征根

$$
Pe^{\alpha t}
$$

如果$\alpha$等于特征根

$$
(P_1t+P_0)e^{\alpha t}
$$

3. 激励为$cos(\beta t)$或$\sin(\beta t)$，如果所有的特征根都不等于$\pm j\beta$

$$
P\cos(\beta t) + Q\sin(\beta t)
$$

这个通常可以把$y_p$直接代入原方程求解系数。

注意，如果题目给的条件是$y(0),y'(0)$等条件，我们需要在全解中去求齐次解中设的参数。如果题目给的条件是$y_{zi},y'_{zi}$等条件，我们在齐次解中求参数即可。

### 系统的初始值

## 用方框图描述LTI

## 奇异函数

$$
x(t) = x(t) * \delta(t)
$$

$$
x'(t) = x(t)*\delta '(t)
$$

$$
x''(t) = x(t) * \delta ''(t) = x(t) * \delta '(t) * \delta '(t)
$$

以此类推。而

$$
x(t) * u(t) = \int^t_{-\infty}x(\tau)d\tau
$$

也可以以此类推。

其中$u(t)*u(t) = tu(t)$称为单位斜坡函数。

此外还有


$$
f(t)\delta(t)=f(0)\delta(t)
$$

$$
\int^{+\infty}_{-\infty}f(t)\delta(t)dt=f(0)
$$

$$
f(t)\delta(t-a) = f(a)\delta(t-a)
$$

$$
\int^{+\infty}_{-\infty}f(t)\delta(t-a)dt = f(a)
$$

$$
f(t)\delta'(t) = f(0)\delta'(t)-f'(0)\delta(t)
$$

$$
\int^{+\infty}_{-\infty}f(t)\delta'(t)dt = -f'(0)
$$

$$
\int^{+\infty}_{-\infty}f(t)\delta'(t-a)dt = -f'(a)
$$

$$
\int^{+\infty}_{-\infty}f(t)\delta^{(n)}(t)dt = (-1)^nf^{(n)}(0)
$$

$$
\delta(at) = \dfrac{1}{|a|}\delta(t)
$$

$$
\delta^n(at) = \dfrac{1}{|a|}\dfrac{1}{a^n}\delta^n(t)
$$

$$
\delta^{(n)}(-t)=(-1)^n\delta^{(n)}(t)
$$

# 周期信号的傅里叶级数表示

## LTI对复指数信号的响应

LTI对复指数信号的响应也是一个复指数信号，不同的只是在幅度上的变化

$$
e^{st}\rightarrow H(s)e^{st}
$$

$$
z^n = H(z)z^n
$$

上面$s,z$是复数，我们更多讨论$s=jw,z=e^{jw}$的情况。

其中$H$是一个复振幅因子。

一个信号，如果系统对该信号的输出响应仅是一个常数（可能是复数）乘以输入，那么这个信号就是这个系统的特征函数，幅度因子称为特征值。

如果将输入信号表示为傅立叶级数

$$
x(t) = \sum_ka_ke^{s_kt}
$$

则响应一定是

$$
y(t) = \sum_k a_kH(s_k)e^{s_kt}
$$

离散情况下

$$
x[n] = \sum_ka_kz^n_k
$$

$$
y[n] = \sum_k a_kH(z_k)z^n_k
$$

## 连续时间周期信号的傅里叶级数表示

$$
x(t) = \sum^{+\infty}_{k=-\infty}a_ke^{jkw_0t} =  \sum^{+\infty}_{k=-\infty}a_ke^{jk(2\pi/T)t}
$$

$k=0$的一项是一个常数，$k=\pm 1$的两项合称为基波分量，也称为一次谐波分量，之后类推。

也可以写作

$$
x(t) = a_0 + 2\sum^\infty_{k=1}[B_k\cos kw_0 t - C_k\sin kw_0t]
$$

其中$a_n$由以下公式确定

$$
a_n = \dfrac{1}{T}\int^T_0x(t)e^{-jnw0t}dt
$$

或者也可以是在任意一个最小正周期内的积分，记作$\int_T$

$a_n$称为傅里叶级数系数或频谱系数，$a_0$为直流或常数分量。

## 傅立叶级数的收敛

**条件1**

在任何周期内，$x(t)$必须绝对可积，即

$$
\int_T|x(t)|dt<\infty
$$

这也保证了平方可积，也就保证了能量有限，每一个系数有限。

**条件2**

在任意有限区间内，$x(t)$具有有限个起伏变化。也就是说，在任何单个周期内，$x(t)$的最大值和最小值的数目有限。

**条件3**

在$x(t)$的任何有限区间内，只有有限个不连续点，而且在这些不连续点上，函数是有限值。

在这些不连续点上，傅里叶级数收敛到不连续点处的平均值。

## 连续时间傅里叶级数的性质

首先设$x(t),y(t)$的周期都为$T$，傅立叶系数为$a_k,b_k$

*线性*

$$
Ax(t)+By(t)\leftrightarrow Aa_k+Bb_k
$$

*时移*

$$
x(t-t_0) \leftrightarrow a_ke^{-jkw_0t_0} = a_ke^{-jk(2\pi/T)t_0}
$$

*频移*

$$
e^{jMw_0t}x(t) = e^{jM(2\pi/T)t}x(t) \leftrightarrow a_{k-M}
$$

*共轭*

$$
x^*(t) = a^*_{-k}
$$

*时间反转*

$$
x(-t) = a_{-k}
$$

*时域尺度变换*

$$
x(\alpha t),\alpha >0(周期为T/\alpha) \leftrightarrow a_k
$$

*周期卷积*

$$
\int_Tx(\tau)y(t-\tau)d\tau\leftrightarrow Ta_kb_k
$$

*相乘*

$$
x(t)y(t) \leftrightarrow a_k*b_k
$$

*微分*

$$
\dfrac{dx(t)}{dt} \leftrightarrow jkw_0a_k=jk\dfrac{2\pi}{T}a_k
$$

*积分*

$$
\int^t_{-\infty} x(t)dt \leftrightarrow \dfrac{1}{jkw_0}a_k = \dfrac{1}{jk(2\pi/T)}a_k
$$

*周期信号的帕塞瓦尔定理*

$$
\dfrac{1}{T}\int_T|x(t)|^2 dt = \sum^{+\infty}_{k=-\infty}|a_k|^2
$$

## 奇次谐波和偶次谐波

TODO

## 离散时间周期信号的傅里叶级数表示

离散时间周期信号的傅里叶级数是有限项级数，而连续情况下是无穷级数。所以离散情况下不存在收敛的问题。

离散情况下，周期N满足

$$
x[n] = x[n+N]
$$

其中最小的正整数$N$对应的角频率称为基波频率。

$x[n]$的傅里叶级数表示为

$$
x[n] = \sum_k a_ke^{jkw_0n} = \sum_ka_ke^{jk(2\pi/N)n}
$$

其中这个求和是，k在任意连续$N$个数中都成立的。所以也可以记作

$$
x[n] = \sum_{k=<N>}a_ke^{jkw_0n}
$$

$a_k$由以下公式确定

$$
a_k = \dfrac{1}{N}\sum_{n=<N>}x[n]e^{-jkw_0n} = \dfrac{1}{N}\sum_{n=<N>}x[n]e^{-jk(2\pi/N)n}
$$

## 离散时间傅里叶级数的性质

整体和连续的情况差别不大。

首先设$x[n],y[n]$的周期都为$N$，傅立叶系数为$a_k,b_k$，系数是周期的，周期也为$N$

*线性*

$$
Ax[n]+By[n]\leftrightarrow Aa_k+Bb_k
$$

*时移*

$$
x[n-n_0] \leftrightarrow a_ke^{-jk(2\pi/N)n_0}
$$

*频移*

$$
e^{jM(2\pi/N)n}x[n]\leftrightarrow a_{k-M}
$$

*共轭*

$$
x^*[n] = a^*_{-k}
$$

*时间反转*

$$
x[-n] = a_{-k}
$$

*时域尺度变换*

$$
x_{(m)}[n] = \begin{cases}
x[n/m] & n是m的倍数\\
0 & n不是m的倍数
\end{cases}
\leftrightarrow \dfrac{1}{m}a_k,周期为mN
$$

*周期卷积*

$$
\sum_{r=<N>}x[r]y[n-r]\leftrightarrow Na_kb_k
$$

*相乘*

$$
x[n]y[n] \leftrightarrow \sum_{l=<N>}a_l b_{k-l}
$$

*差分*

$$
x[n]-x[n-1] \leftrightarrow (1-e^{-jk(2\pi/N)})a_k
$$

*求和*

$$
\sum^n_{k=-\infty}x[n](仅当a_0=0才为有限的且为周期的)\leftrightarrow \bigg(\dfrac{1}{1-e^{-jk(2\pi/N)}}\bigg)a_k
$$

*周期信号的帕塞瓦尔定理*

$$
\dfrac{1}{N}\sum_{n=<N>}|x[n]|^2 = \sum_{k=<N>}|a_k|^2
$$

## 傅里叶级数与LTI

若$x(t)=e^{st}$是一个连续时间线性时不变系统的输入，其输出久违$y(t)=H(s)e^{st}$，而

$$
H(s) = \int^{+\infty}_{-\infty} h(\tau)e^{-s\tau}d\tau
$$

其中$h(\tau)$是该线性时不变系统的单位冲激响应。

同理离散情况下$x[n]=z^n$，$y[n]=H(z)z^n$

$$
H(z) = \sum^{+\infty}_{k=-\infty}h[k]z^{-k}
$$

当$s,z$是一般复数时，$H(s),H(z)$就称为该系统的系统函数。一般我们考虑$s=jw,z=e^{jw}$，此时$H(s),H(z)$称为频率响应。

若用傅里叶级数表示输入

$$
x(t) = \sum^{+\infty}_{k=-\infty}a_ke^{jkw_0t}
$$

则输出为

$$
y(t) = \sum^{+\infty}_{k=-\infty}a_kH(jkw_0)e^{jkw_0t}
$$

此时$y(t)$也是周期的，并且与$x(t)$有相同的基波频率。若$a_k$是$x(t)$的傅立叶系数，那么$a_kH(jkw_0)$就是$y(t)$的傅里叶系数。

## 滤波

TODO

# 连续时间傅里叶变换

## 非周期信号的表示

可以把非周期信号当成一个周期信号在周期任意大时的极限。

设$\tilde x(t)$是一个周期信号，而$x(t)$是这个周期信号的其中一个周期，其他部分全为$0$，那么（方便起见取周期$[-T/2,T/2]$）

$$
a_k = \dfrac{1}{T}\int^{T/2}_{-T/2}x(t)e^{-jkw_0t}dt = \dfrac{1}{T}\int^{+\infty}_{-\infty}x(t)e^{-jkw_0t}dt
$$

定义$Ta_k$的包络$X(jw)$为

$$
X(jw) = \int^{+\infty}_{-\infty}x(t)e^{-jwt}dt
$$

则

$$
a_k= \dfrac{1}{T}X(jkw_0)
$$

然后我们就可以用上式去表示$\tilde{x}(t)$，当周期无限大时，表示$x(t)$如下

$$
x(t) = \dfrac{1}{2\pi}\int^{+\infty}_{-\infty}X(jw)e^{jwt}dw
$$

$$
X(jw) = \int^{+\infty}_{-\infty}x(t)e^{-jwt}dt
$$

这两个式子称为傅立叶变换对。$X(jw)$称为$x(t)$的傅里叶变换，也称为$x(t)$的频谱。

## 傅立叶变换的收敛

和之前一样

1. $\int^{+\infty}_{-\infty}|x(t)|dt<\infty$
2. 任何有限区间内，$x(t)$只有有限个最大值和最小值
3. 任何有限区间内，$x(t)$有有限个不连续点，并且在每个不连续点处都必须是有限值。

## 周期信号的傅立叶变换

$$
x(t) = \sum^{+\infty}_{k=-\infty}a_ke^{jkw_0t}
$$

$$
X(jw) = \sum^{+\infty}_{k=-\infty}2\pi a_k\delta(w-kw_0)
$$

## 连续时间傅里叶变换的性质

设非周期信号$x(t),y(t)$，傅立叶变换分别为$X(jw),Y(jw)$

*线性*

$$
ax(t)+by(t)\leftrightarrow aX(jw)+bY(jw)
$$

*时移*

$$
x(t-t_0)\leftrightarrow e^{-jwt_0}X(jw)
$$

*频移*

$$
e^{jw_0t}x(t)\leftrightarrow X(j(w-w_0))
$$

*共轭*

$$
x^*(t)\leftrightarrow X^*(-jw)
$$

*时间反转*

$$
x(-t)\leftrightarrow X(-jw)
$$

*尺度变换*

$$
x(at)\leftrightarrow\dfrac{1}{|a|}X(\dfrac{jw}{a})
$$

*卷积*

$$
x(t)*y(t)\leftrightarrow X(jw)Y(jw)
$$

*相乘*

$$
x(t)y(t)\leftrightarrow \dfrac{1}{2\pi} X(jw)*Y(jw)
$$

*时域微分*

$$
\dfrac{d}{dt}x(t)\leftrightarrow jwX(jw)
$$

*积分*

$$
\int^t_{-\infty}x(t)dt\leftrightarrow \dfrac{1}{jw}X(jw)+\pi X(0)\delta(w)
$$

*频域微分*

$$
tx(t)\leftrightarrow j\dfrac{d}{dw}X(jw)
$$

*非周期信号的帕塞瓦尔定理*

$$
\int^{+\infty}_{-\infty}|x(t)|^2dt=\dfrac{1}{2\pi}\int^{+\infty}_{-\infty}|X(jw)|^2dw
$$

*对偶*

$$
X(jt)\leftrightarrow 2\pi x(-w)
$$

## 基本傅立叶变换对

$$
\sum^{+\infty}_{k=-\infty}a_k e^{jkw_0t}\leftrightarrow 2\pi\sum^{+\infty}_{k=-\infty}a_k\delta(w-kw_0)
$$

$$
e^{jkw_0t}\leftrightarrow 2\pi\delta(w-kw_0)
$$

$$
cos(w_0t)\leftrightarrow\pi[\delta(w-w_0)+\delta(w+w_0)]
$$

$$
sin(w_0t)\leftrightarrow \dfrac{\pi}{j}[\delta(w-w_0)-\delta(w+w_0)]
$$

$$
x(t)=1\leftrightarrow 2\pi\delta(w)
$$

$$
g_\tau(t)\leftrightarrow tSa(\dfrac{wt}{2})
$$

$$
sgn(t)\leftrightarrow \dfrac{2}{jw}
$$

$$
\delta(t)\leftrightarrow 1
$$

$$
u(t)\leftrightarrow\dfrac{1}{jw}+\pi\delta(w)
$$

其中

$sgn(t)$是符号函数，$Sa(\theta)=sin\theta/\theta$，$g_\tau(t)$代表门函数，即在$[-\tau/2,\tau/2]$的范围内取$1$，其他范围内取$0$。
