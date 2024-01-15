---
title: 干货 | VirtualBox打开虚拟机VERR_INVALID_NAME(-104)错误的解决
date: 2023-10-22T22:33:16+08:00
draft: false
tags:
  - VirtualBox
  - Windows
  - 干货
categories: 其他计算机科学
markup: goldmark
---

可能是因为我安装了Debian子系统，VBox突然就打不开了。当然时间久远，一直没有解决，是不是这个原因暂且按下不表。

报错信息大致是

```
...
Error relaunching VirtualBox VM process: 5
...
5 VERR_INVALID_NAME (-104) 
-Invalid (malformed) file/path name.
```

这样的。

我知道很长一段时间内，Hyper-v和VirtualBox、VMWare是不兼容的。但是现在VBox都已经7.0版本了，甚至都支持半虚拟化用Hyper-v了，再用hyper-v和virtualbox不兼容来解释就有点说不过去了。

经过一番搜索，我的方法如下：

1. 在VBox的安装目录下找到`\drivers\vboxsup`，选中`VBoxSup.inf`右键安装。
2. 打开`regedit.exe`，找到`计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\VBoxSup`，把`Start`改成`2`
3. 重启

竟然奇迹般的好了。

很多人也通过别的方法解决了，例如关闭杀毒软件、关闭Hyper-V等等。我个人觉得这个错误的原因很多，可能需要自己多看几篇文章，反复尝试。
