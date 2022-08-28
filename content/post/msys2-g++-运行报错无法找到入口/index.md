---
title: "Msys2中使用mingw64的G++编译，运行报错无法找到入口"
date: 2022-08-28T16:18:50+08:00
draft: false
tags: [MSYS2]
categories: 其他
mathjax: false
---

![1.jpg](1.jpg)

情况如图所示，_ZSt28__throw_bad_array_new_lengthv无法被定位。

经过检查，通过mingw64.exe运行的g++版本是12.1.0。这也是通过[官网的教程](https://www.msys2.org/)安装的版本。并且我们通常会把msys2\mingw64\bin\作为环境变量。

而通过msys2.exe运行的g++版本是11.3.0。将环境变量换为msys2\usr\bin\，此时编译的程序可以正常运行不报错。

推测原因是版本问题，或者是msys2和mingw64有什么差别。

另外，通过mingw64打开的命令行来运行编译出来的exe文件不会报错。

