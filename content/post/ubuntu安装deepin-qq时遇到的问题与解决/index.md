---
title: ubuntu安装deepin-qq时遇到的问题与解决
date: 2021-01-03 12:16:26
image: "cover.jpg"
tags: 
    - Ubuntu
    - Linux
    - QQ
categories:
    - Linux
description: 关于 libgirepository-1.0-1 破坏 python-gi (< 3.34.0-4~) 但是 3.30.4-1 正要被安装 的报错
---

系统：Ubuntu 20.10

今天使用Ubuntu，想安装一下deepin的qq，在网上找到以下方法：

```
wget -O- https://deepin-wine.i-m.dev/setup.sh | sh
```

正常执行

```
sudo apt-get install com.qq.im.deepin
```

<!--more-->

报错：

```
下列软件包有未满足的依赖关系：
 libgirepository-1.0-1 : 破坏: python-gi (< 3.34.0-4~) 但是 3.30.4-1 正要被安装
E: 无法修正错误，因为您要求某些软件包保持现状，就是它们破坏了软件包间的依赖关系。
```

我试着安装python-gi，同样报错，我又试着删了libgirepository-1.0-1，但是他是很多包的依赖，不敢删。

百度搜索无果，bing搜索外国也没找到解决办法，倒是有人遇到了同样的问题。

在ubuntu搜集信息后，发现libgirepository-1.0-1依赖于libffi7，但是apt下载不到他，只能去https://packages.ubuntu.com/zh-cn/focal/libffi7手动下载。安装完后又去https://packages.ubuntu.com/zh-cn/focal/python-gi手动下载python-gi，先后安装成功。

再次安装qq，重启，安装成功。

但是发现字体显示不全。找了个网站下载了simsun.ttc，放到~/.deepinwine/Deepin-QQ/drive_c/windows/Fonts/

问题解决。

![图片 1](ubuntu安装deepin-qq时遇到的问题与解决/1.jpg)


后记：现在linux下qq有官方支持的新版本了，应该没人再用这个了吧。


