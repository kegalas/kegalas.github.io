---
title: "MSYS2报错David Macek is unknown trust"
date: 2022-07-01T12:05:59+08:00
draft: false
tags: [MSYS2]
categories: 其他
---

在MSYS2中安装程序时遇到如下问题：

```
error: mingw-w64-x86_64-mpc: signature from "David Macek <david.macek.0@gmail.com>" is unknown trust
```

尝试通过[https://packages.msys2.org/](https://packages.msys2.org/package/msys2-keyring?repo=msys&variant=x86_64)中提供的方法，安装了key，解决问题。

或者直接输入pacman -S msys2-keyring
