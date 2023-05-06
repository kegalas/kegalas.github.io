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

1. 如果$2\pi/\beta$是整数，则周期为$N=2\pi/\beta$
2. 如果$2\pi/\beta$是有理数，则周期为$N=M(2\pi/\beta)$，$M$为使$N$称为正整数的最小正整数
3. 如果$2\pi/\beta$是无理数，则不是周期信号。

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

单位冲激的导数，即单位冲激偶有

$$
\delta'(t) = \begin{cases}
 \pm\infty & \text{ if } t=0 \\
 0 & \text{ if } t\neq 0
\end{cases}
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

一个系统对脉冲信号的（零状态）响应记作$h$。一个系统的特征可以完全由脉冲响应刻画，一个系统对其他信号的响应可以用该信号和脉冲响应的卷积表示。

$$
y[n] = x[n]*h[n]
$$

$$
y(t) = x(t)*h(t)
$$

通常，题目中不会给以下两个条件。因为是零状态的响应，所以$h(0_-)=h'(0_-)=0$

## 线性时不变系统的性质

### 线性时不变下卷积的性质

1. 交换律
2. 分配律
3. 结合律

当然他都叫线性时不变系统了，肯定有线性和时不变性。

**微分特性（仅限于连续情况）**

$$
\dfrac{d^n}{dt^n}[f_1*f_2] = \dfrac{d^n f_1}{dt^n}*f_2 = f_1*\dfrac{d^n f_2}{dt^n}
$$

**积分特性（仅限于连续情况）**

$$
\int^t_{-\infty}[f_1*f_2]d\tau = [\int^t_{-\infty}f_1d\tau]*f_2 = f_1 * [\int^t_{-\infty}f_2d\tau]
$$

**$f_1(-\infty)=0$或$f_2'(\infty) = 0$（仅限连续情况）**

$$
f_1*f_2=f_1'*f_2'
$$

**（后向）差分特性（仅限于离散情况）**

$$
\nabla[f_1*f_2] = \nabla f_1*f_2 = f_1*\nabla f_2
$$

**时移特性**

若$f(t)=f_1(t)*f_2(t)$

则

$$
f_1(t-t_1)*f_2(t-t_2) = f_1(t-t_1-t_2)*f_2(t) = f_1(t)*f_2(t-t_1-t_2) = f(t-t_1-t_2)
$$

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

## 用微分方程描述的连续因果线性时不变系统

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

3. 如果是一对共轭复根$\lambda_{1,2} = \alpha\pm j\beta$，则

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

4. 激励为$\delta,\varepsilon$等时，由于我们的特解是零状态响应中的部分，所以都是在$0_+$时的取值，我们直接把激励在$0_+$的值写在等号右边，然后设定$y_p=p$（常数），解出$p$即可。显然，$p$的值会是$\varepsilon\text{的系数}/y\text{的系数}$（因为$y$现在是常数，任意阶导数为$0$）。如果是$e^t\varepsilon(t)$这种形式又怎么办呢，还是老话，我们求特解是$0_+$时的值，此时看做激励为$e^t$，然后用上面的特解。

注意，如果题目给的条件是$y(0),y'(0)$等条件，我们需要在全解中去求齐次解中设的参数。如果题目给的条件是$y_{zi},y'_{zi}$等条件，我们在齐次解中求参数即可。

如果给的激励包含多种情况？我自己的思考是，求出冲激响应即可，然后根据线性性质，将激励分别与冲激响应卷积，再相加。

还有一种办法是求频率响应$H(jw)$来求零状态响应，见后。

### 系统的初始值

初始值是$n$阶系统在$t=0$时接入激励，其响应在$t=0_+$时刻的值，即$y^{(j)}(0_+) (j=0,1,2,\cdots,n-1)$。

初始状态是指系统在激励尚未接入的$t=0_-$时刻的响应值$y^{(j)}(0_-)$，该值反映了系统的历史情况，而与激励无关。

例如$y''(t)+3y'(t)+2y(t)=2\delta(t)+6\varepsilon(t)$，（老师在PPT中给出但我没有看懂的办法是）两端积分

$$
\int^{0_+}_{0_-}y''(t)dt+3\int^{0_+}_{0_-}y'(t)dt+2\int^{0_+}_{0_-}y(t)dt = 2\int^{0_+}_{0_-}\delta(t)dt + 6\int^{0_+}_{0_-}\varepsilon(t)dt
$$

其中从左到右每一项分别对应，$y'(0_+)-y'(0_-),y(0_+)-y(0_-),0,2,0$

所以$y'(0_+)-y'(0_-) = 2, y(0_+)-y(0_-) = 0$

结论是，微分方程等号右边含有$\delta(t)$及其各阶导数时，响应$y(t)$及其各阶导数由$0_-$到$0_+$的瞬间将发生跃变（如果不含，则$y(t)$在t=0处是连续的，$y(0_+)=0$）。这时可按如下步骤由$0_-$求取$0_+$（以二阶系统为例）

1. 将$f(t)$代入微分方程。如果等号右边含有$\delta(t)$及其各阶导数，根据微分方程等号两端各奇异函数的系数相等原理，判断方程左端$y(t)$的最高阶导数所含$\delta(t)$导数的最高阶次。
2. 令$y''(t) = a\delta''(t)+b\delta'(t)+c\delta(t)+r_0(t)$（其中$r_0(t)$是指不含$\delta(t)$及其各阶导数的项，在第四步中的积分，由于不含这些项，所以是$0$），对$y''(t)$进行积分（从$-\infty$到$t$），求得$y'(t),y(t)$。
3. 将$y'',y',y$代入微分方程，根据等号两端各奇异函数的系数相等，从而求得各个待定系数。
4. 分别对$y',y''$等号两端从$0_-$到$0_+$积分，求得$0_+$值

用这种方法能对上式较好的求解。

必须要提到的一点是，设$g(f) = \int^{0_+}_{0_-}f(x)dx$，应该只有$f(x)=\delta(x)$（不考虑常数系数），才有$g(f)=1$，其他全部都有$g(f)=0$。或许有其他函数也满足$g(f)\neq 0$，但是在信号这里，所有连续的函数、阶跃函数、冲激函数的导数、二阶导数、$n$阶导数$g(f)$都等于$0$，只有冲激函数自己等于$1$。

### 零输入响应

记作$y_{zi}$，对应齐次微分方程的解。故不存在跃变。

### 零状态响应

记作$y_{zs}$，对应非齐次方程的解，需要求出来齐次解和特解，再加起来。最后的全响应需要将零输入和零状态加起来。注意零输入和零状态都要求一个齐次解，虽然它们形式相同，但是系数可能不同。

有一种投机取巧的办法求全响应就是用电路课中介绍的三要素法。

### 响应分类

**响应可以分为固有响应和强迫响应**

固有响应仅与系统本身的特性有关，而与激励的函数形式无关。是微分方程的齐次解。

强迫响应与激励的函数形式有关。是微分方程的特解。

**响应可以分为暂态响应和稳态响应**

暂态响应是指响应中暂时出现的分量，随着时间的增长，它将消失。

稳态响应是稳定的分量，若存在，通常表现为阶跃函数和周期函数。比如，电路系统中的直流稳态响应和正弦稳态响应。

### 微分方程求冲激响应

例如我们有$y''+a_1y'+a_0y' = b_2f''+b_1f'+b_0f$，我们可以分两步进行

1. 选择一个响应$h_1(t)$，使得

$$
h_1''+a_1h_1'+a_0h_1 = \delta
$$

$$
h_1(0_-)=h_1'(0-) = 0
$$

计算出$h_1(0_+),h_1'(0_+)$，然后我们使用之前提到的解法求出$h_1$，不过显然这里特解为$0$，和齐次解具有同样的形式。

2. 根据线性性质和微分特性，可知$h = b_2h_1''+b_1h_1'+b_0h_1$。不过注意这个$b_i$是常数，如果我们等式右边是$b_2f''+b_0f+q$，这个$q$不是$f$的导数（其他情况类似），则我们的$h=b_2h_1''+b_0h_1+q*h_1$。当然你也可以全部都做卷积，只是不是把$b_2,b_1,b_0$拿来做卷积，而是把整体的$b_2f'',b_1f',b_0f$拿来做卷积。如果$f=h$，卷积的结果就是$b_2h_1''+b_1h_1'+b_0h_1$

### 微分方程求阶跃响应

类似的，还是上面那个例子

1. 选择一个响应$g_1(t)$，使得

$$
g_1''+a_1g_1'+a_0g_1 = \varepsilon
$$

$$
g_1(0_-)=g_1'(0-) = 0
$$

计算出$g_1(0_+),g_1'(0_+)$，然后我们使用之前提到的解法求出$g_1$

2. 根据线性性质和微分特性，可知$g = b_2g_1''+b_1g_1'+b_0g_1$

或者，如果我们已知冲激响应，我们有更简单的办法，由于微积分特性，我们可以求出

$$
g(t) = \int^t_{-\infty} h(\tau)d\tau
$$

同理

$$
h(t) = g'(t)
$$

## 用差分方程描述的离散因果线性时不变系统

### 差分方程

一般形式为

$$
y[k]+a_{n-1}y[k-1]+\cdots+a_0y[k-n] = b_mf[k]+\cdots+b_0f[k-m]
$$

和微分方程相似，差分方程的解也可以分为$y = y_h+y_p$

齐次解也是由齐次方程求特征根得到。

对于每一个$\lambda_i$，

1. 如果它是单实根，则

$$
y_h = C\lambda^k
$$

2. 如果它是二重实根，则

$$
y_h = (C_1k+C_0)\lambda^k
$$

3. 如果是一对共轭复根$\lambda_{1,2} = \alpha\pm j\beta$，则

$$
\rho^k[C\cos(\beta k)+D\sin(\beta k)]
$$

整个齐次解就是把这些对于每一个$\lambda_i$的齐次解加起来，再根据条件求出待定系数。

特解的常见形式为

1. 激励为$f[k]=k$时，如果所有的特征根均不等于$1$，

$$
P_1 k+P_0
$$

如果有$1$个等于$1$的特征根，

$$
k(P_1k+P_0)
$$

2. 激励为$f[k]=a^k$，如果$a$不等于特征根

$$
Pa^k
$$

如果$a$等于特征根

$$
(P_1k+P_0)a^k
$$

3. 激励为$cos(\beta k)$或$\sin(\beta k)$，如果所有的特征根都不等于$e^{\pm j\beta}$

$$
P\cos(\beta k) + Q\sin(\beta k)
$$

这个通常可以把$y_p$直接代入原方程求解系数。

4. 激励为$\delta,\varepsilon$时。如果只有$\varepsilon$，则我们直接设置$y[k]$为常数，与连续的情况不同，差分不会使得$y[k-1]$等于零。解出的特解就是$y$的各项系数之和分之$\varepsilon$的系数之和（我确定这个在激励只有$\varepsilon[k]$的时候是成立的，如果有$\varepsilon[k-1]$等东西成不成立我不确定，没找到网上和教材中讨论这个）。如果只有$\delta$，则实际上是在求冲激响应，由后面可得，就是在求齐次解形式的解，不存在特解。如果两个都有，只管$\varepsilon$（这是我的猜测，没找到教材讨论这个，我们或许还可以考虑$\delta[k]=\varepsilon[k]-\varepsilon[k-1]$）。

注意，如果题目给的条件是$y[k],y[k-1]$等条件，我们需要在全解中去求齐次解中设的参数。如果题目给的条件是$y_{zi}[k],y_{zi}[k-1]$等条件，我们在齐次解中求参数即可。

### 零输入响应

同前

### 零状态响应

同前

### 差分方程求单位脉冲响应

例如$y[k]-4y[k-1]+3y[k-2]=3f[k]-f[k-1]$，我么可以首先

$$
h[k]-4h[k-1]+3h[k-2]=3\delta[k]-\delta[k-1]
$$

由隐含条件（二阶下），$h[-1]=h[-2]=0$，又由上式，我们可以求出$h[0],h[1]$。再对上式用传统方法求出齐次解。

由于代入求解参数时代入的是$h[0],h[1]$，对于$k=1,k=0$时方程也成立（实际上单位脉冲响应和零输入响应、齐次解具有同样的形式，不需要特解），所以单位脉冲响应就是$h[k] = [-1+4(3)^k]\varepsilon[k]$

当然，如果等式右边出现的是$f[k-2]+f[k-1]$这样的，我们就要求出$h[2],h[1]$，然后用这个代入求解齐次解的参数。

或者我们还是用老办法，求出$h_1[k]-4h_1[k-1]+3h_1[k-2]=\delta[k]$，中$h_1[k]$的表达式（需要用$h[0],h[-1]$求出参数），然后在根据线性性质求出$h[k]$

### 差分方程求单位阶跃响应

阶跃响应和差分方程的全解是一个形式。我们求出全解即可。注意隐含条件（二阶下）是$g[-1]=g[-2]=0$，我们可以求出$g[0],g[1]$，代入求出参数。

作为区分，激励为$\delta[k]$时要求出$h[0],h[-1]$，激励为$\varepsilon[k]$时要求出$h[1],h[0]$

或者由差分特性

$$
g[k] = \sum^k_{i=-\infty}h[i]
$$

同理有

$$
h[k] = g[k] - g[k-1]
$$

## 用方框图描述LTI

## 奇异函数

**对于连续情况**

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

**对于离散情况**

$$
f[k]*\delta[k] = f[k]
$$

$$
f[k]*\delta[k-k_0] = f[k-k_0]
$$

$$
f[k]*\varepsilon[k] = \sum^k_{i=-\infty}f[i]
$$

此外还有

$$
f[k]\delta[k] = f[0]\delta[k]
$$

$$
f[k]\delta[k-k_0] = f[k_0]\delta[k-k_0]
$$

$$
\sum^{+\infty}_{k=-\infty}\delta[k] = 1
$$

$$
\sum^{+\infty}_{k=-\infty}f[k]\delta[k] = f[0]
$$

$$
\sum^{+\infty}_{k=-\infty}f[k]\delta[k-k_0] = f[k_0]
$$

$$
\sum^{k}_{i=-\infty}\delta[i] = \varepsilon[k]
$$

$$
\delta[k] = \varepsilon[k]-\varepsilon[k-1]
$$

$$
\delta[k] = \delta[-k]
$$

## 相关函数

实函数$f_1(t)$和$f_2(t)$，如为能量有限信号，它们之间的互相关函数定义为：

$$
R_{12}(\tau) = \int^{+\infty}_{-\infty}f_1(t)f_2(t-\tau)dt = \int^{+\infty}_{-\infty}f_1(t+\tau)f_2(t)dt
$$

$$
R_{21}(\tau) = \int^{+\infty}_{-\infty}f_1(t-\tau)f_2(t)dt = \int^{+\infty}_{-\infty}f_1(t)f_2(t+\tau)dt
$$

一般两者不相等，但是有$R_{12}(\tau) = R_{21}(-\tau)$

如果$f_1,f_2$是同一个函数，那么无须区分二者，直接用$R$表示自相关系数，

$$
R(\tau) = \int^{+\infty}_{-\infty}f(t+\tau)f(t)dt = \int^{+\infty}_{-\infty}f(t)f(t-\tau)dt
$$

此时$R(\tau)=R(-\tau)$

这东西和卷积的形式很像，显然我们可以推出来

$$
R_{12} = f_1(t)*f_2(-t)
$$

# 周期信号的傅里叶级数表示

## 信号的正交分解

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

或者

$$
x(t) = \dfrac{a_0}{2}+\sum^\infty_{n=1}a_n\cos(n\Omega t)+\sum^\infty_{n=1}b_n\sin(n\Omega t) = \dfrac{A_0}{2} + \sum^\infty_{n=1}A_n\cos(n\Omega t+\varphi_n)
$$

其中（指数表示的那个）$a_n$由以下公式确定

$$
a_n = \dfrac{1}{T}\int^T_0x(t)e^{-jnw_0t}dt
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

## 谐波特性

**偶函数**

如果$x(t)$是偶函数，则其没有正弦分量，只有余弦分量和直流分量。

**奇函数**

如果$x(t)$是奇函数，则其没有余弦分量和直流分量，只有正弦分量。

**奇谐函数**

傅里叶级数中只含有奇次谐波分量，不含有偶次谐波分量，即

$$
a_0=a_2=a_4=\cdots=b_2=b_4=\cdots=0
$$

表现为$x(t) = -x(t\pm T/2)$

**偶谐函数**

傅里叶级数中只含有偶次谐波分量，不含有奇次谐波分量，即

$$
a_1=a_3=\cdots=b_1=b_3=\cdots=0
$$

表现为$x(t) = x(t\pm T/2)$

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

若$x(t)=e^{st}$是一个连续时间线性时不变系统的输入，其输出就为$y(t)=H(s)e^{st}$，而

$$
H(s) = \int^{+\infty}_{-\infty} h(\tau)e^{-s\tau}d\tau
$$

其中$h(\tau)$是该线性时不变系统的单位冲激响应（这其实是$h(t)$的傅立叶变换）。

同理离散情况下$x[n]=z^n$，$y[n]=H(z)z^n$

$$
H(z) = \sum^{+\infty}_{k=-\infty}h[k]z^{-k}
$$

当$s,z$是一般复数时，$H(s),H(z)$就称为该系统的系统函数。一般我们考虑$s=jw,z=e^{jw}$，此时$H(s),H(z)$称为频率响应。

此时我们可以说

若用傅里叶级数表示输入

$$
x(t) = \sum^{+\infty}_{k=-\infty}a_ke^{jkw_0t}
$$

则输出为

$$
y(t) = \sum^{+\infty}_{k=-\infty}a_kH(jkw_0)e^{jkw_0t}
$$

此时$y(t)$也是周期的，并且与$x(t)$有相同的基波频率。若$a_k$是$x(t)$的傅立叶系数，那么$a_kH(jkw_0)$就是$y(t)$的傅里叶系数。

离散情况下，若用傅里叶级数表示输入

$$
x[n] = \sum_{k=<N>} a_ke^{jk(2\pi/N)n}
$$

那么输出为

$$
y[n] = \sum_{k=<N>} a_kH(e^{j2\pi k/N})e^{jk(2\pi/N)n}
$$

$y[n]$也是周期的，且和$x[n]$具有相同的周期，$y[n]$的第$k$个傅里叶系数就是$a_kH(e^{j2\pi k/N})$

## 周期信号的频谱

分为单边谱和双边谱。

单边谱是对于三角函数形式展开的，

$$
x(t) = \dfrac{A_0}{2} + \sum^\infty_{n=1}A_n\cos(n\Omega t+\varphi_n)
$$

频谱分为两个函数图像，即$A_n\sim\omega$的幅度谱和$\varphi_n\sim\omega$的相位谱（$\omega=n\Omega,n=0,1,2,\cdots$）

双边谱是对于指数形式展开的，

$$
x(t) = \sum^{+\infty}_{k=-\infty}a_ke^{jkw_0t} 
$$

频谱分为两个函数图像，即$|a_k|\sim\omega$的幅度谱和$\varphi_k\sim\omega$的相位谱（$\omega=k\omega_0,\varphi_k=k\omega_0 t,k=0,\pm1,\pm2,\cdots$）

## 滤波

按我的理解，简单来说，就是弄一个系统，使得它的频率响应$H(jw)$弄成你想要滤波的效果。

比如，你想要弄一个理想低通滤波器，那么你就设计一个系统，使得其

$$
H(jw) = \begin{cases}
 1 & \text{ if } |w|\leq w_c \\
 0 & \text{ if } |w| >w_c
\end{cases}
$$

这样输出$y(t)=H(jw)x(t)$，就只会保留$x(t)$中频率在$[-w_c,w_c]$之间的部分。

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

但有些函数其实不满足，也可以收敛，有傅里叶变换，比如阶跃函数。

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

如果$x(t)$为实函数，$x(-t)\leftrightarrow X(-jw) = X^*(jw)$

如果$x(t)$为虚函数$jg(t)$，$x(-t)\leftrightarrow X(-jw) = -X^*(jw)$

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
\dfrac{d^n}{dt^n}x(t)\leftrightarrow (jw)^nX(jw)
$$

*积分*

$$
\int^t_{-\infty}x(t)dt\leftrightarrow \dfrac{1}{jw}X(jw)+\pi X(0)\delta(w)
$$

*频域微分*

$$
tx(t)\leftrightarrow j\dfrac{d}{dw}X(jw)
$$

$$
(-jt)^nx(t)\leftrightarrow X^{(n)}(jw)
$$

*频域积分*

$$
\dfrac{1}{-jt}x(t)+\pi x(0)\delta(t)\leftrightarrow \int^w_{-\infty}X(jx)dx
$$

*非周期信号的帕塞瓦尔定理*

$$
\int^{+\infty}_{-\infty}|x(t)|^2dt=\dfrac{1}{2\pi}\int^{+\infty}_{-\infty}|X(jw)|^2dw
$$

*对称*

$$
X(jt)\leftrightarrow 2\pi x(-w)
$$

*相关定理*

前面我们介绍过相关函数，有

$$
R_{12}(t) = x_1(t)*x_2(-t)
$$

$$
R_{21}(t) = x_2(t)*x_1(-t)
$$

$$
R(t) = x(t)*x(-t)
$$

我们有

$$
R_{12}(t)\leftrightarrow X_1(jw)X_2^*(jw)
$$

$$
R_{21}(t)\leftrightarrow X_2(jw)X_1^*(jw)
$$

$$
R(\tau)\leftrightarrow |X(jw)|^2
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
g_\tau(t)\leftrightarrow \tau Sa(\dfrac{w\tau}{2})
$$

$$
sgn(t)\leftrightarrow \dfrac{2}{jw}
$$

$$
\delta(t)\leftrightarrow 1
$$

$$
\delta^{(n)}\leftrightarrow(jw)^n
$$

$$
u(t)\leftrightarrow\dfrac{1}{jw}+\pi\delta(w)
$$

$$
e^{-\alpha t}\varepsilon(t)\leftrightarrow\dfrac{1}{\alpha+jw},\alpha>0
$$

$$
e^{-\alpha |t|}\leftrightarrow\dfrac{2\alpha}{\alpha^2+w^2},\alpha>0
$$

其中

$sgn(t)$是符号函数，$Sa(\theta)=sin\theta/\theta$，$g_\tau(t)$代表门函数，即在$[-\tau/2,\tau/2]$的范围内取$1$，其他范围内取$0$。

阶跃函数此时可以表示为$\varepsilon(t) = \dfrac{1}{2}[1+sgn(t)]$

## 线性常系数微分方程表征的系统

线性常系数微分方程有如下形式

$$
\sum^N_{k=0}a_k\dfrac{d^k y(t)}{dt^k} = \sum^m_{k=0}b_k\dfrac{d^k x(t)}{dt^k} 
$$

我们可以像之前一样用经典方法去解输入对应的输出（零状态响应）。但是我们也可以使用频率响应来求出。

如果我们有$x(t)=e^{jwt}$，则系统的输入就是$y(t)=H(jw)e^{jwt}$。

一个事实是，$H(jw)\leftrightarrow h(t)$。我们之前还说过，$y(t)=h(t)*x(t)$，我们就可以得到$Y(jw)=H(jw)X(jw)$。

或者我们有

$$
H(jw) = \dfrac{Y(jw)}{X(jw)} = \dfrac{\sum^M_{k=0}b_k(jw)^k}{\sum^{N}_{k=0}a_k(jw)^k}
$$

其中第二个等号后面的式子可以把微分方程的两边全部取傅立叶变换，再提项得到。

有了$H(jw)$，我们就容易求出对于任意一个输入信号$f(t)$，这个系统的零状态响应。

要么我们可以$Y(jw)=H(jw)F(jw)$再傅立叶逆变换得到$y(t)$，要么$y(t)=H(jw)f(t)$。

$H(jw)$一般是复函数，记为

$$
H(jw)=|H(jw)|e^{j\theta(w)}
$$

其中$|H(jw)|$称为幅频特性或者幅频响应，是$w$的偶函数。

$\theta(w)$称为相频特性或相频响应，是$w$的奇函数。

## 帕塞瓦尔能量方程

$$
E = \lim_{T\to\infty}\int^T_{-T}|f(t)|^2dt = \int^\infty_{-\infty}|f(t)|^2dt = \dfrac{1}{2\pi}\int^\infty_{-\infty}|F(jw)|^2dw
$$

## 能量频谱

$$
\mathcal{E}(w) = |F(jw)|^2
$$

根据前面的相关定理，我们可以得到

$$
R(\tau)\leftrightarrow \mathcal{E}(w)
$$

即能量信号的自相关函数与其能量谱是一对傅里叶变换。

## 频带宽度

在满足一定失真条件下，信号可以用某段频率范围的信号来表示，此频率范围称为频带宽度。

信号频谱的距离原点最近的两个零点之间集中了信号的绝大部分能量。信号的功率集中在低频段。

1. 一般把第一个零点作为信号的频带宽度
2. 对于一般周期信号，将幅度下降为$\dfrac{1}{10}|F_n|_{\text{max}}$的频率定义为频带宽度
3. 系统的通频带$>$信号的带宽，才能不失真。

## 功率频谱

$$
P = \lim_{T\to\infty}\dfrac{1}{T}\int_T f^2(t)dt = \dfrac{1}{2\pi}\int^\infty_{-\infty}\lim_{T\to\infty}\dfrac{|F_T(jw)|^2}{T}dw
$$

记

$$
\mathcal{P}(w) = \lim_{T\to\infty}\dfrac{|F_T(jw)|^2}{T}
$$

也可以证明

$$
R(\tau)\leftrightarrow\mathcal{P}(w)
$$

即功率信号的自相关函数与其功率谱是一对傅里叶变换。

# 离散时间傅里叶变换

# 信号与系统的时域和频域特性

# 采样

## 冲激串采样

周期冲激串$p(t)$称为采样函数，周期$T$称为采样周期，而$p(t)$的基波频率$w_s=2\pi/T$称为，采样频率。

在时域中有

$$
x_p(t) = x(t)p(t)
$$

其中

$$
p(t) = \sum^{+\infty}_{n=-\infty}\delta(t-nT)
$$

经过计算可以得到，频域中有

$$
X_p(jw) = \dfrac{1}{T}\sum^{+\infty}_{k=-\infty}X(j(w-kw_s))
$$

## 采样定理

设$x(t)$是某一个带限信号，在$|w|>w_M$时，$X(jw)=0$。如果$w_s>2w_M$，其中$w_s=2\pi /T$，那么$x(t)$就唯一地由其样本$x(nT),n=0,\pm1,\pm2,\cdots$所确定

在采样定理中，采样频率必须大于$2w_M$，该频率$2w_M$一般称为奈奎斯特速率。

已知样本值，重建$x(t)$的方法：产生一个周期冲激串，其冲激幅度就是这些依次而来的样本值；然后将该冲激串通过一个增益为$T$，截止频率大于$w_M$而小于$w_s-w_m$的理想低筒滤波器，该滤波器的输出就是$x(t)$。

这个滤波器的$h(t)$为

$$
h(t) = T_s\dfrac{w_c}{\pi}Sa(w_ct)
$$

其中$w_M<w_c<w_s-w_M$，方便起见常取$w_c=0.5w_s$

此时

$$
x(t) = \sum^{+\infty}_{n=-\infty}x(nT_s)Sa[\dfrac{w_s}{2}(t-nT_s)]
$$

# 拉普拉斯变换

一个信号$x(t)$的拉普拉斯变换定义如下

$$
X(s) = \int^{+\infty}_{-\infty}x(t)e^{-st}dt
$$

当然，傅立叶变换就是$s=jw$时的一个特例。
