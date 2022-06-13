---
title: "大学物理公式整理"
date: 2022-06-13T19:13:12+08:00
draft: false
tags: [物理,大学]
description: 大学物理相关的公式（可能还有一些概念）整理
categories: 物理
mathjax: true
markup: pandoc
---

[TOC]

## 运动学

### 位移

位移：$\Delta\bm{r}$

位移的大小：$|\Delta\bm{r}|$

位矢大小的增量：$|\Delta r|$

其中后两者一般是不相等的，不能搞混。

### 速度

1. 平均速度

$$
\overline{\bm{v}}=\frac{\bm{r}(t+\Delta t)-\bm{r}(t)}{\Delta t}=\frac{\Delta\bm{r}}{\Delta t}
$$

2. 平均速度的大小

$$
|\overline{\bm{v}}|=\left|\frac{\Delta\bm{r}}{\Delta t}\right|
$$

同样的一般有

$$
|\overline{\bm{v}}|\neq\left|\frac{\Delta r}{\Delta t}\right|
$$

3. 瞬时速度

$$
\bm{v}=\frac{d\bm{r}}{dt}
$$

速度的大小常称速率。

$$
|\bm{v}|=\left|\frac{d\bm{r}}{dt}\right|
$$

同样一般有

$$
|\bm{v}|\neq \left|\frac{dr}{dt}\right|
$$

### 加速度

1. 平均加速度

$$
\overline{\bm{a}}=\frac{\Delta\bm{v}}{\Delta t}
$$

2. 瞬时加速度

$$
\bm{a}=\frac{d\bm{v}}{dt}
$$

关于加速度的大小和一般不相等与（）的性质类似于速度，不再介绍。

### 直角坐标表示运动

其位移、速度、加速度都可以分成几个坐标分量来计算，总的位移、速度、加速度则是勾股定理的形式，不再介绍。

### 自然坐标法表示运动

$$
\bm{v}=\frac{ds}{dt}\bm{\tau}
$$

其中$\bm{\tau}$是切向量。

$$
\bm{a}_n=a_n\bm{n}=\frac{v^2}{r}\bm{n}
$$

其中$n$是法向量，$r$是曲率半径，曲率半径计算见高数上整理。

$$
\bm{a}_\tau=a_\tau\bm{\tau}=\frac{dv}{dt}\bm{\tau}
$$

$$
\bm{a=a}_n+\bm{a}_\tau
$$

### 圆周运动的角量表示

1. 角坐标

$$
\theta=\theta(t)
$$

2. 角速度和角平均速度

$$
\overline{\omega}=\frac{\Delta\theta}{\Delta t}
$$

$$
\omega=\frac{d\theta}{dt}
$$

3. 角加速度和角平均加速度

$$
\overline{\beta}=\frac{\Delta\omega}{\Delta t}
$$

$$
\beta=\frac{d\omega}{dt}=\frac{d^2\theta}{dt^2}
$$

4. 线速度、加速度与角速度的关系

$$
v=r\omega\\
a_\tau=r\beta\\
a_n=r\omega^2
$$

### 坐标系变换

$$
\bm{v}_a=\bm{v}_r+\bm{u}
$$

即绝对速度等于相对于坐标系的速度与坐标系的绝对速度的矢量和。

$$
\bm{a}_a=\bm{a}_r+\bm{a}_e
$$

类似。

### 牛顿运动定律

1. 第一定律

$$
R=\sum_i\bm{F}_i=0
$$

也可以将$\bm{F}$写成坐标分量的形式。

2. 第二定律

$$
\bm{R}=\sum_i\bm{F}_i=\frac{d(m\bm{v})}{dt}
$$

质量为常量时

$$
\bm{R}=m\frac{d\bm{v}}{dt}=m\bm{a}
$$

可以写作坐标分量和切向量、法向量分量的形式。

3. 第三定律

$$
\bm{F}_1=\bm{F}_2
$$

### 刚体的平动

任意时刻，平动刚体上个点的速度、加速度都相同。

## 力学

### 常见的几种力

1. 万有引力

$$
\bm{F}_{21}=-G\frac{m_1m_2}{r^2}\bm{r}^0\\
G=6.67\times10^{-11}\quad m^2/(kg\cdot s^2)
$$

2. 弹性力

$$
F_x=-kx
$$

3. 摩擦力
   
**静摩擦力**

$$
f_{max}=\mu_0N
$$

前者为静摩擦系数，后者为支持力。

**滑动摩擦力**

$$
f=\mu N
$$

前者为滑动摩擦系数。

### 力矩

$$
M_O=\bm{r}\times\bm{F}
$$

单位：$N\cdot m$

### 转动惯量

$$
J_z=\int_Vr^2dm
$$

### 常见物体的转动惯量计算公式

![5.2](5.2.jpg)

### 平行轴定理

![5.11](5.11.jpg)

$$
J_z'=J_z+Mh^2
$$

### 转动惯量和力矩的关系

$$
M_z=J_z\beta
$$

## 功和能

### 功

1. 恒力做功

$$
A=\bm{F}\cdot\bm{s}=Fscos\theta
$$

2. 变力做功

$$
A=\int^b_{a(L)}\bm{F}\cdot d\bm{r}
$$

通常会拆分成对坐标系求曲线积分。

3. 平均功率

$$
P=\frac{\Delta A}{\Delta t}
$$

4. 瞬时功率

$$
P=\frac{dA}{dt}
$$

$$
P=\frac{\bm{F}\cdot d\bm{r}}{dt}=\bm{F}\cdot \bm{v}=Fvcos\theta
$$

### 几种常见力的功

1. 重力的功

$$
A=mg(z_1-z_2)
$$

2. 万有引力的功

$$
A=GmM(\frac{1}{r_2}-\frac{1}{r_1})
$$

3. 弹性力的功

$$
A=\frac{1}{2}k\lambda_1^2-\frac{1}{2}k\lambda_2^2
$$

### 动能定理

1. 质点动能定理

$$
dA=d(\frac{1}{2}mv^2)
$$

$$
A=\frac{1}{2}mv_1^2-\frac{1}{2}mv_2^2
$$

2. 质点系动能定理

$$
\sum_i A_i=E_{k2}-E_{k1}
$$

### 势能、机械能守恒定律

1. 保守力

做功只与始末位置有关而与路径无关的力。

2. 势能

零势能点$M_0$，空间中的某个点$M$

$$
E_p=\int_M^{M_0}\bm{F}\cdot d\bm{r}
$$

3. 重力势能

$$
E_p=mgz
$$

4. 万有引力势能

$$
E_p=-G\frac{mM}{r}
$$

5. 弹性势能

$$
A=\frac{1}{2}kx_1^2-\frac{1}{2}kx_2^2
$$

### 绕定轴转动刚体的动能、动能定理

1. 动能

$$
E=\frac{1}{2}J_z\omega^2
$$

2. 力矩的功

$$
A=\int_{\theta_1}^{\theta_2}M_z(\bm{F})d\theta
$$

3. 动能定理

$$
A=\frac{1}{2}J_z\omega_2^2-\frac{1}{2}J_z\omega_1^2
$$

## 冲量、动量、角动量

### 质点系动量定理

$$
d(m\bm{v})=\bm{F}dt
$$

$$
\bm{I}=m\bm{v}_2-m\bm{v}_1=\int^{t_2}_{t_1}\bm{F}dt
$$

如果是恒力

$$
m\bm{v}_2-m\bm{v}_1=\bm{F}(t_2-t_1)
$$

### 质点系动量定理

$$
\sum_i m_i\bm{v}_i-\sum_i m\bm{v}_{i0}=\sum_i\int^{t}_{t_0}\bm{F}_i dt
$$

### 质点系动量守恒定律

作用在质点系上的所有外力的矢量和为零，则该质点系的动量保持不变。

如果某个方向的矢量和为零，则这个方向上的动量保持不变。

$$
\sum_i m_i\bm{v}_{ix}=C
$$

$C$是常量

### 质心、质心运动定理

1. 质心位置

见高数下整理

2. 质心运动定理

质点系质心的运动，可以看成为一个质点的运动，这个质点集中了整个质点系的质量，也集中了质点系收到的所有外力。

### 动量矩和动量矩守恒定律

1. 动量矩

$$
\bm{L}_O=\bm{r}\times m\bm{v}
$$

$$
L_z=J_z\omega
$$

2. 动量矩定理

$$
\frac{d\bm{L_O}}{dt}=\bm{r}\times\bm{F}=\bm{M}_O
$$

3. 动量矩守恒定律

当作用在质点上的合理对固定点之矩总是为零时，质点动量对该点的矩为常矢量。即

$$
\bm{M}_O=0\Rightarrow \bm{L}_O=\bm{C}
$$

$\bm{C}$是常矢量。

4. 刚体绕定轴转动的动量矩定理

$$
(J_z\omega)_t-(J_z\omega)_{t_0}=\int^t_{t_0}M_zdt
$$

5. 刚体绕定轴转动的动量矩守恒定律

$$
M_z=0\Rightarrow J_z\omega=C
$$

## 机械振动

### 简谐振动

$$
x=Acos(\omega t+\varphi)
$$

$$
v=\overset{\cdot}{x}=-A\omega sin(\omega t+\varphi)
$$

$$
a=\overset{\cdot\cdot}{x}=-A\omega^2 cos(\omega t+\varphi)
$$

对于弹簧振子的周期：

$$
T=\frac{2\pi}{\omega}=2\pi\sqrt{\frac{m}{k}}
$$

对于单摆的周期：

$$
T=2\pi\sqrt{\frac{l}{g}}
$$

### 弹簧串联并联和弹性系数

1. 串联

$$
k=\frac{k_1k_2}{k_1+k_2}
$$

2. 并联

$$
k=k_1+k_2
$$

注：有一种两根弹簧中间连了物体的，是一种并联。

### 谐振动的能量

$$
E=\frac{1}{2}kA^2
$$

一个周期内，动能和势能的平均大小：

$$
\overline{E_p}=\frac{1}{4}kA^2
$$

$$
\overline{E_k}=\frac{1}{4}kA^2
$$

### 谐振动的合成

1. 同方向，同频率的合成

频率不变

$$
A=\sqrt{A_1^2+A_2^2+2A_1A_2cos(\varphi_2-\varphi_1)}
$$

$$
\varphi = arctan\frac{A_1sin\varphi_1+A_2sin\varphi_2}{A_1cos\varphi_1+A_2cos\varphi_2}
$$

2. 同方向不同频率的合成

$$
A=\sqrt{A_1^2+A_2^2+2A_1A_2cos(\omega_2-\omega_1)t}
$$

$$
\tau=\frac{2\pi}{|\omega_2-\omega_1|}
$$

$$
\nu=\frac{|\omega_2-\omega_1|}{2\pi}=|\nu_2-\nu_1|
$$

$\nu$为拍频。

3. 两个相互垂直谐振动的合成

根据参数方程求出平面解析式。

## 机械波

### 机械波的产生和传播

拉紧的绳子，横波的波速为

$$
u_t=\sqrt{\frac{T}{\mu}}
$$

其中$T$是绳子的张力，$\mu$是线密度。

### 平面简谐波

1. 波函数

正向传播：

$$
y(x,t)=Acos\left[\omega\left(t-\frac{x}{u}\right)+\varphi_0\right]
$$

负向传播：

$$
y(x,t)=Acos\left[\omega\left(t+\frac{x}{u}\right)+\varphi_0\right]
$$

### 波的能量

1. 能量

设绳子每单位长度的质量为$\mu$，线元总机械能：

$$
W=W_k+W_p=\mu\Delta xA^2\omega^2sin^2\left[\omega\left(t-\frac{x}{u}\right)+\varphi_0\right]
$$

2. 能量密度

把单位体积中波的能量称为波的能量密度：

$$
w=\frac{W}{\Delta V}=\frac{W}{\Delta x\Delta S}=\rho A^2\omega^2sin^2\left[\omega\left(t-\frac{x}{u}\right)+\varphi_0\right]
$$

3. 能流密度

$$
I=\overline{w}u
$$

$$
I=\frac{1}{2}\rho A^2\omega^2u
$$

$$
\bm{I}=\overline{w}\bm{u}
$$

4. 平面波和球面波的振幅

平面简谐波在理想无吸收的、均匀媒质中传播时振幅不变。

球面波在均匀、无吸收媒质中传播，有

$$
\frac{A_1}{A_2}=\frac{r_2}{r_1}
$$

即该点的振幅和到波源的距离成反比

5. 波的吸收

$$
I=I_0e^{-ax}
$$

### 波的干涉

干涉条件：频率相同、振动方向相同、相位差恒定。

$$
A^2=A_1^2+A_2^2+2A_1A_2cos\Delta\varphi
$$

$$
I=I_1+I_2+2\sqrt{I_1I_2}cos\Delta\varphi
$$

其中上面两式中

$$
\Delta\varphi=(\varphi_2-\varphi_1)-2\pi\frac{r_2-r_1}{\lambda}
$$

如果两个波源的初相位相同，则$\Delta\varphi$只取决于波程差$\delta=r_1-r_2$，于是干涉相长的条件为：

$$
\delta=r_1-r_2=\pm k\lambda,\quad k=0,1,2,\cdots
$$

干涉相消的条件为：

$$
\delta=r_1-r_2=\pm (2k+1)\frac{\lambda}{2},\quad k=0,1,2,\cdots
$$

### 驻波

1. 形成驻波的条件：

$$
L=n\frac{\lambda}{2},\quad n=1,2,3,\cdots
$$

2. 驻波波函数

$$
y=2Acos2\pi\frac{x}{\lambda}\cdot cos2\pi\nu t
$$

### 多普勒效应

1. 波源$S$静止，观察者相对于波源的速度为$v_O$，靠近为正值，远离为负值。则观察者接收到的频率为：

$$
\nu=(1+\frac{v_O}{u})\nu_0
$$

2. 观察者静止，波源相对于观察者的速度为$v_S$，靠近为正值，远离为负值。则观察者接收到的频率为：

$$
\nu=\frac{u}{u-v_S}\nu_0
$$
