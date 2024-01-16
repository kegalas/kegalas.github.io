---
title: 干货 | 添加msys2到右键菜单
date: 2024-01-15T22:40:43+08:00
draft: false
tags:
  - 计算机
  - MSYS2
  - Windows
  - 开发环境
  - 干货
categories: 其他计算机科学
markup: goldmark
---

参考自[https://gist.github.com/elieux/ef044468d067d68040c7](https://gist.github.com/elieux/ef044468d067d68040c7)

但是他那个时候还没有ucrt64，所以我略作修改如下。

首先是添加注册表项：

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Classes\Directory\Background\shell\MSYS here\command]
@="G:\\Program_Files\\msys64\\msys2.exe bash"

[HKEY_CURRENT_USER\Software\Classes\Directory\Background\shell\UCRT64 here\command]
@="G:\\Program_Files\\msys64\\ucrt64.exe bash"

[HKEY_CURRENT_USER\Software\Classes\Directory\Background\shell\MINGW64 here\command]
@="G:\\Program_Files\\msys64\\mingw64.exe bash"

[HKEY_CURRENT_USER\Software\Classes\Directory\Background\shell\CLANG64 here\command]
@="G:\\Program_Files\\msys64\\clang64.exe bash"
```

这里面的路径记得改成自己的安装路径。

然后是删除

```reg
Windows Registry Editor Version 5.00

[-HKEY_CURRENT_USER\Software\Classes\Directory\Background\shell\MSYS here]
[-HKEY_CURRENT_USER\Software\Classes\Directory\Background\shell\UCRT64 here]
[-HKEY_CURRENT_USER\Software\Classes\Directory\Background\shell\MINGW64 here]
[-HKEY_CURRENT_USER\Software\Classes\Directory\Background\shell\CLANG64 here]
```

当然，我自己觉得这样比较丑。因为右键菜单是会按照字母顺序排序的，这样这四个就不在一起了。我个人是想加一个二级菜单把它们放一起的，这个TODO。
