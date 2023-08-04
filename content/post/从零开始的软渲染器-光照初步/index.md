---
title: "从零开始的软渲染器 光照初步"
date: 2023-08-04T21:25:38+08:00
draft: true
tags: [图形学,渲染]
categories: 图形学
mathjax: true
markup: pandoc
image: cover.jpg
---

<u>**[导航页面](../从零开始的软渲染器-导航/)**</u>

用**[这个链接](https://github.com/kegalas/oar/blob/5f4cd5fc90df31b357b3580cf063b4bc83ad779a/src/main.cpp)** 中的代码（需要去下载模型文件放到代码中的指定文件夹，见**[链接](https://github.com/kegalas/oar/blob/1eddb36577cd9403dfc0c763c8a738d21c2bd59c/obj/african_head.obj)** ），我们可以得到如下的效果图

![1.jpg](1.jpg)

看上去还好，细看的话发现眼镜、嘴巴、耳朵一塌糊涂，这实际上是因为三角形的前后关系导致覆盖而出现的结果，在下一部分我们将介绍Z-buffer算法来解决这个问题。
