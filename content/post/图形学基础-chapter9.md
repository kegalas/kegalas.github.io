---
title: "计算机图形学基础（第五版）-第九章笔记"
date: 2022-07-03T15:21:12+08:00
draft: false
tags: [图形学]
categories: 图形学
mathjax: true
markup: pandoc
---

## 光栅化

### 线段绘制

对于一条给定线段

$$
f(x,y)\equiv(y_0-y_1)x+(x_1-x_0)y+x_0y_1-x_1y_0=0
$$

假设$x_0\leq x_1$，否则交换两个点。

设

$$
m=\frac{y_1-y_0}{x_1-x_0}
$$

下面的讨论建立在$m\in (0,1]$上，其他的取值情况类似。

给出伪代码如下

```
y=y0

for x=x0 to x1 do
    draw(x,y)
    if(some condition) then
        y = y+1

```

其中$x,y$都取整数。

一种判断$y=y+1$的方法是，假设当前画出来的点的坐标（而不是序号下标）是$(x,y)$（根据本书的规则，左下角的像素的中心点是原点），则下一个要画的点只有两种情况，要么是右边的点，要么是右上角的点。即$(x+1,y),(x+1,y+1)$，我们取中点，即$(x+1,y+0.5)$，如果直线在这个点的下方，则画右边的点；如果直线在这个点的上方，则画右上方的点。

如果直线在上方（或者点在下方），那么$f(x+1,y+0.5)<0$，在下方则大于0.如果刚好等于零，可以任意画。

所以可以写伪代码如下

```
if f(x+1,y+0.5)<0 then
    y=y+1
```

对于这个算法有优化的办法。主要是针对每次都要调用$f$计算来进行的优化。

注意到我们可能计算过$f(x-1,y+0.5),f(x-1,y-0.5)$，并且我们有

$$
f(x+1,y)=f(x,y)+(y_0-y_1)
$$

$$
f(x+1,y+1) = f(x,y)+(y_0-y_1)+(x_1-x_0)
$$

有如下伪代码

```
y = y0
d = f(x0+1,y0+0.5)
for x = x0 to x1 do
    draw(x,y)
    if d<0 then
        y = y+1
        d = d+(x1-x0)+(y0-y1)
    else
        d = d+(y0-y1)
```
### 三角形绘制

**高洛德插值(Gouraud Interpolation)**

运用重心坐标系，我们可以对颜色进行插值。

假设三个点的颜色值分别为$\bm c_0,\bm c_1,\bm c_2$。假设我们要绘制的点的重心坐标为$(\alpha,\beta,\gamma)$，则其颜色为

$$
\bm c = \alpha\bm c_0+\beta\bm c_1+\gamma\bm c_2
$$

**暴力光栅化算法**

伪代码如下

```
for all x do
    for all y do
        compute(alpha,beta,gamma) for (x,y)
        if(alpha,beta,gamma in [0,1]) then
            c = alpha*c0+beta*c1+gamma*c2
            drawpixel(x,y) with color c
```

**优化后的算法**

```
xMin = floor(xi)
xMax = ceiling(xi)
yMin = floor(yi)
yMax = ceiling(yi)
for y = yMin to yMax do
    for x = xMin to xMax do
        alpha = f12(x,y)/f12(x0,y0)
        beta = f20(x,y)/f20(x1,y1)
        gamma = f01(x,y)/f01(x2,y2)
        if(alpha,beta,gamma>0) then
            c=alpha*c0+beta*c1+gamma*c2
            drawpixel(x,y) with color c
```

其中的$f_{ij}$为

$$
f_{01}(x,y) = (y_0-y_1)x+(x_1-x_0)y+x_0y_1-x_1y_0\\
f_{12}(x,y) = (y_1-y_2)x+(x_2-x_1)y+x_1y_2-x_2y_1\\
f_{20}(x,y) = (y_2-y_0)x+(x_0-x_2)y+x_2y_0-x_0y_2
$$

**对于公共边上的点的处理**

```
xMin = floor(xi)
xMax = ceiling(xi)
yMin = floor(yi)
yMax = ceiling(yi)

fAlpha = f12(x0,y0)
fBeta = f20(x1,x1)
fGamma = f01(x2,y2)

for y = yMin to yMax do
    for x = xMin to xMax do
        alpha = f12(x,y)/fAlpha
        beta = f20(x,y)/fBeta
        gamma = f01(x,y)/fGamma
        if(alpha,beta,gamma>=0) then
            if(alpha>0 or fAlpha*f12(-1,-1)>0)and
            (beta>0 or fBeta*f20(-1,-1)>0)and
            (gamma>0 or fGamma*f01(-1,-1)>0) then
                c=alpha*c0+beta*c1+gamma*c2
                drawpixel(x,y) with color c
```


