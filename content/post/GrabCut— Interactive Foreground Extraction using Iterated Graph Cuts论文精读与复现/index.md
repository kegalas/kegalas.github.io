---
title: GrabCut— Interactive Foreground Extraction Using Iterated Graph Cuts论文精读与复现
date: 2024-05-20T11:21:12+08:00
draft: true
tags:
  - 大学
  - 人工智能
  - 图形学
  - 计算机视觉
categories: 图形学
mathjax: true
markup: goldmark
image: cover.jpg
---

# 摘要

分离出图像的前景和背景，对于图像编辑是很重要的。经典的image segmentation实现方式有两个，一是使用纹理（颜色）信息，例如魔棒工具。另一种是使用边缘信息，例如磁力套索（Intelligent Scissors）。本文介绍使用graph-cut来进行图像分割。本文提出了三方面的改进：优化迭代、优化用户交互、border matting的处理优化

# 简介

这里提到，所谓的“前景对象”的结果是一个alpha-matte，反应了前景和背景的比例。个人理解应该是，某个像素属于前景的可能性的大小和背景的可能性的比。本文想要实现保证高性能的前提下，不需要用户的过多交互，就能得到很好的效果。

高性能指：精准分割前背景，主观上可信的alpha值，可以处理模糊、混合像素（mixed pixels）、和透明度（透明的物体），前景图片不带有背景透过来的颜色。

用户交互的简易程度：作者举例了两个极端，一个是需要用户逐像素地点击前景，一个是只需要用户点击前景和背景的几个位置。

## 之前的交互式获取matte的工作

![1.jpg](1.jpg)

**魔棒（Magic Band）**

和PS里用到的魔棒差不多，就是选择容差，选择图片中点击区域附近在容差之类的像素。但是这个容差有时不容易选择。导致结果不好。

**磁力套索（Intelligent Scissors）**

和PS里的磁力套索差不多，用户需要在前景的边缘大概地移动鼠标，选取整个前景。缺点是如果背景太复杂，用户需要的操作会很多。

**Bayes matting**

对颜色概率分布进行建模，来实现一个full alpha mattes。用户要选取一个trimap。记作$T=\{T_B,T_U,T_F\}$，其中$T_B$标记了背景，$T_F$标记了前景。之后alpha值是在剩下的那个区域$T_U$上计算的。有时效果还是不错的，但是只有当$T_U$区域不太大，并且前背景的颜色概率分布可以很好地分开时，效果才不错。另外，需要的用户操作也不少。

**Knockout 2**

是一个PS的插件，里面用的是用户定义的trimap，总体和Bayes matting差不多。

**Graph Cut**

优化了bayes matting，同样利用了trimaps和对颜色的概率分布建模，鲁棒性很好，它甚至可以区分出环境里的迷彩服。具体在第二节才会介绍。

**Level sets**

通过解偏微分方程来做。缺点是可能会陷入局部最优。

## 作者提出的系统：grabcut

理想状态中，一个matting工具应该能够在$T_U$上计算连续的alpha值，没有强制约束的话，alpha的值可能就只会是0、1两个值。如果有连续的alpha值，那么对于烟雾、头发、树（叶）等问题应该可以自动地、近似地解决。但是作者发现，对于前景背景颜色分布容易分开的图像效果还可以，遇到迷彩这样的东西就不行。 

首先作者会在第二节和第三节介绍一个使用迭代图割（iterative graph cut）的方法来获取图像硬分割（hard segmentation）。然后在第四节border matting部分会用上通过硬分区之间的区域来计算alpha value。作者说，grabcut不处理完全透明的情况，除非是在border部分。作者建议使用matting brush来处理这种完全透明，只要没有迷彩，结果就可以很好。

作者认为，本文的首要创新在于处理图像的分割。他们为graph cut提供了两个优化：迭代估计、不完全标记。这两个东西可观的减少了用户交互的复杂度。只需要拖拽一个矩形框柱前景。另外，也开发出了一种新的计算alpha值的机制，来用于border matting的部分。

# 使用Graph Cut进行图像分割

grabcut是建立在graph cut的研究基础上的，所以有必要对此进行详细描述。

## 图像分割

graph cut是用于黑白图像分割的。给定一个初始的trimap，$T$，图像是一个数组$z=(z_1,\cdots,z_N)$，每个值都是灰度值，下标是一维的。图像的分割用一个“不透明度”的数组来表示，$\underline{\alpha}=(\alpha_1,\cdots,\alpha_N)$表示。每个像素都有一个这种数组。一般来说$\alpha_n$是$[0,1]$上的实数，但对于硬分割来说，其取值为集合$\{0,1\}$。$0$代表背景，$1$代表前景。

参数$\underline{\theta}$描述图像前景和背景的grey-level概率分布，也包含了灰度值的直方图。即

$$
\underline{\theta} = \{h(z;\alpha), \alpha=0,1\}
$$

其中直方图是直接用$T_B, T_F$标记的像素得来的。这里直方图进行了归一化，$\int_zh(z;\alpha)=1$

图像分割的任务就是使用给定的$z$和$\underline{\theta}$，来推断出未知部分的不透明度数组$\underline{\alpha}$

## 最小化能量法来进行分割

定义能量函数$E$，其值最小化的时候，得到最好的分割结果。某种意义上，这个$E$应该和前景、背景的直方图都有关，并且也和不透明度有关。反应了物体的一种稳定趋势。作者从吉布斯自由能中获得了灵感，写出了一个类似的形式：

$$
E(\underline{\alpha},\underline{\theta},z) = U(\underline{\alpha},\underline{\theta},z)+V(\underline{\alpha},z)
$$

其中，数据项$U$在给定直方图模型$\underline{\theta}$的情况下，评估不透明度分布$\underline{\alpha}$对图像$z$的拟合。定义为

$$
U(\underline{\alpha},\underline{\theta},z) = \sum_n-\log h(z_n;\alpha_n)
$$

其中，平滑项为

$$
V(\underline{\alpha},z) = \gamma\sum_{(m,n)\in C}dis(m,n)^{-1}[\alpha_n\neq\alpha_m]\exp-\beta(z_m-z_n)^2
$$

里面的$[...]$是一个艾弗森括号。$C$是一个集合，里面的元素是邻居像素点对。$dis(\cdot)$是邻居像素点对的欧拉距离。这种能量（指平滑项）让灰度相似的区域更加具有一致性。实践中，像素的邻居有8个。

当常数$\beta=0$时，平滑项就简化为著名的Ising prior（但我不知道是什么）。此时平滑项就对全图进行平滑，其程度由$\gamma$决定。在Graph Cut的论文中指出，设置$\beta>0$更好，这可以放松对于高对比度区域的平滑，同时这里的$\beta$可以选为

$$
\beta = \bigg(2\bigg<(z_m-z_n)^2\bigg>\bigg)^{-1}
$$

这里$<\cdot>$代表图像上样本的期望。选这个$\beta$能够保证，平滑项可以适当的从高对比度和低对比度场景中切换。

常数$\gamma$取$50$，这是后面的文章通过实验得到的经验值。

在定义完能量模型后，图像分割就可以被估计为

$$
\underline{\hat{\alpha}} = \arg\min_{\underline{\alpha}} E(\underline{\alpha}, \underline{\theta})
$$

这里的最小化使用最小割算法。这也是硬分割的基础。

下一部分会介绍作者三方面的改进。首先是把黑白图像的模型，用高斯混合模型替换，从而可以用到彩色图像上。第二，执行一次最小割的算法被替换为一个迭代过程。第三，简化用户操作，只需要使用一个矩形或者一个套索来标注$T_B$

# GrabCut图像分割算法

这一部分也是硬分割的。

## 对彩色图像数据建模

现在图像是RGB三通道的了，再像以前一样去构建灰度直方图是不切实际的（从256变成了17M）。

作者这里参考前人的工作，对前景和背景各使用一个GMM，其$K$值都等于$0$。简便起见，在优化框架框架中，添加了一个向量$k=\{k_1,\cdots,k_n,\cdots,k_N\}$，其中$k_n\in\{1,\cdots,K\}$。为每个像素分配一个不同的GMM分量，根据$\alpha_n=0$还是$1$来确定分配前景还是背景GMM分量。

此时，吉布斯能公式变为

$$
E(\underline{\alpha},k,\underline{\theta},z) = U(\underline{\alpha},k,\underline{\theta},z)+V(\underline{\alpha},z)
$$

现在数据项$U$变为

$$
U(\underline{\alpha},k,\underline{\theta},z) = \sum_nD(\alpha_n,k_n,\underline{\theta},z_n)
$$

$$
D(\alpha_n,k_n,\underline{\theta},z_n) = -\log p(z_n|\alpha_n,k_n,\underline{\theta})-\log\pi(\alpha_a,k_n)
$$

其中$p$是正态分布，$\pi$是高斯混合模型中的那个混合系数。展开有

$$
D(\alpha_n,k_n,\underline{\theta},z_n) = -\log\pi(\alpha_n,k_n)+\dfrac{1}{2}\log\det\Sigma(\alpha_n, k_n)+\dfrac{1}{2}[z_n-\mu(\alpha_n, k_n)]^T\Sigma(\alpha_n, k_n)^{-1}[z_n-\mu(\alpha_n, k_n)]
$$

此时模型的参数为

$$
\underline{\theta} = \{\pi(\alpha, k), \mu(\alpha, k), \Sigma(\alpha, k), \alpha=0,1,k=1,\cdots,K\}
$$

平滑项几乎没变

$$
V(\underline{\alpha},z) = \gamma\sum_{(m,n)\in C}[\alpha_n\neq\alpha_m]\exp-\beta||z_m-z_n||^2
$$

这里的距离变成了颜色空间中的欧拉距离。

## 能量最小化迭代法来进行图像分割

GrabCut使用了迭代的算法来替代Graph Cut的一次性算法。好处在于我们可以通过迭代来不断地精炼不透明度$\underline{\alpha}$。然后对于初始的trimap中的$T_U$部分的新标记，被用于精炼GMM的参数$\underline{\theta}$。算法见下图

![2.jpg](2.jpg)

首先是初始化部分

- 用户用矩形框框选前景，那么方框外的东西就是$T_B$了，而方框内的东西都是$T_U$，即还未确定是否是$T_F$
- 对$T_B$内的每个像素初始化$\alpha_n=0$，对$T_U$内的每个像素初始化$\alpha_n=1$
- 然后我们就得到了属于背景的像素和属于前景的像素，这时候就可以初始化GMM的参数了。首先分别对前景和背景的像素，分别用k-means聚类为$K$类，即得到GMM中的$K$个高斯模型的元素。然后对每个高斯模型的元素计算均值、协方差，然后混合系数也可以通过计算属于第$k$个高斯模型的元素占全体元素的比例来得到。

迭代过程：

1. 对每个像素分配GMM中的高斯分量。$k_n$即为使得$D_n$最小的那个$k_n$，公式见上图
2. 对于给定的图像$z$，学习优化GMM的参数。
3. 使用最小割来进行图像分割。
4. 重复步骤1-3直到收敛。作者指出，由于能量是一直递减的，所以一定会收敛。至少会收敛到局部最优。
5. 应用第四节提到的border matting

用户编辑过程：

- 编辑：人为地固定一些像素为背景或前景，来更新trimap，然后执行一次第3步。
- 重操作：（可选）重复整个迭代过程。

## 用户人工编辑

GrabCut是根据颜色分布和边缘来进行图像分割的，但是也有一些“不合常理”的图片。此时还是需要进行人工的编辑。

**Incomplete trimaps**

这里指的应该就是，框选前景，反过来标记背景的操作。整个迭代算法也是不对$T_B$进行retract，但是$T_U$中的像素就可以被改变为背景或前景。

**Further user editing**

即前面描述的用户编辑过程。如果分割的图像错误地把一些东西分到了背景，则我们可以用前景刷把他们标记回来。反之亦然。

# 透明度处理

一个matting的工具应该可以处理连续的$\alpha$值。作者提出了一种机制，来使得之前的硬分隔增强为软分割，从而更好地处理物体边缘。

## Border Matting



# 我的实现
