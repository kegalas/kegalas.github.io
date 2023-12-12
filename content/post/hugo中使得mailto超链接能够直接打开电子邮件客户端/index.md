---
title: "Hugo的Stack皮肤中使得mailto超链接能够直接打开电子邮件客户端"
date: 2022-06-25T23:46:57+08:00
draft: false
tags: [Hugo,计算机]
description: 通过修改Stack的配置文件，使得mailto超链接能够直接打开电子邮件客户端，而不是在浏览器中打开一个奇怪的链接。
categories: 其他计算机科学
mathjax: true
image: "cover.jpg"
---


以Hugo站点为根目录，首先将\themes\hugo-theme-stack\layouts\partials\sidebar\中的left.html复制到\layouts\partials\sidebar\中。

然后修改复制后的文件，如下图。

![1.jpg](1.jpg)

第41行中选中的部分原来是relLangURL，改成absURL。

不过这个方法是否会导致其他问题还有待观察。
