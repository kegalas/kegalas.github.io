---
title: "MSYS2中的make工具安装方法"
date: 2023-05-13T15:15:55+08:00
draft: false
tags: [MSYS2,计算机,Windows,MinGW]
categories: 其他计算机科学
mathjax: false
---

你可能会遇到如下情况：无论你使用MSYS2还是MinGW64还是Ucrt64还是Clang64，去输入pacman -S make，你的make都只会安装在/usr/bin里，也就是MSYS2里。如果你安装它们的toolchain，例如pacman -S mingw-w64-ucrt-x86_64-toolchain，里面包含了mingw-w64-ucrt-x86_64-make这个工具，但是在/ucrt64/bin里面并没有make.exe。

其实并不是没有安装，只不过，他叫mingw32-make.exe。你如果要像使用make一样使用这个工具，你可以在打开cmd，在/ucrt64/bin中，输入`mklink make mingw32-make.exe`来创建软连接。这样，如果这个目录在环境变量之下，你就可以使用ucrt64中的make工具了。

值得注意的是，虽然他叫mingw32-make，不过编译32位和64位的源码都可以用。
