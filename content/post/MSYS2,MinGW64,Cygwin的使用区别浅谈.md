---
title: "MSYS2,MinGW64,Cygwin的使用区别浅谈"
date: 2023-05-06T23:44:47+08:00
draft: false
tags: [MSYS2,计算机,Windows,MinGW]
categories: 其他计算机科学
mathjax: false
---

这篇文章不探讨这几个东西的历史、关系，只来说一下什么时候该用什么，以及区别。

首先，在MSYS2官网上下载安装后，它会提供许许多多的exe文件，包括msys2.exe, clang64.exe, mingw64.exe, ucrt64.exe等。这些东西里面都提供gcc或者clang/llvm等编译器，当然也提供其他一些工具链，但是编译出来的目标文件却不相同。我们把这些东西分为两类。

第一类是msys2.exe，它的目标文件依赖于msys2提供的一个叫做msys-2.0.dll的动态链接库，这是一个虚拟POSIX环境。通过这个东西，一个程序可以调用Linux的API，但是经过这个dll后，转换到调用Windows对应的API来实现。程序本身不是原生的目标文件，而是中间加了一个仿真。

第二类是其他三个，它们的目标文件是原生Windows的，也就是说，不需要这个msys-2.0.dll，直接就可以打开用。缺点是如果你要开发的东西非常依赖于POSIX，则这个东西没法编译，只能去用MSYS2。

而Cygwin，和MSYS2差不多（非常简略地来说的话），只不过POSIX环境变成了cygwin.dll。

总结，如果你需要给别人发Release，那么用MinGW、UCRT、CLANG的编译器（当然你也可以使用MSYS2然后附上msys-2.0.dll）。如果你开发的东西非常依赖POSIX，那么用MSYS2。

至于Cygwin，由于它的安装和卸载我不是很喜欢，我一般不用。不过Cygwin有它很优秀的X server适配，在Windows下表现很好，这点比MSYS2好（实际上它根本没有X server和一些图形库）。不过随着WSL2的发布，也不是必需品了。

现在MSYS2官网更推荐使用UCRT而不是MINGW。如果你要使用clang/llvm，那么用clang64是肯定的（我试了，在clang64里面装用pacman -S clang装进的是MSYS2里面。而且版本比较旧，也没有clangd等工具。我们要用的是 pacman -S mingw-w64-clang-x86_64-toolchain，当然也可以用[llvm-mingw](https://github.com/mstorsjo/llvm-mingw/releases)，不过好像有一些其他问题。而且这个clang64的前端好像是gcc，而不是在Windows下安装的llvm的默认前端msvc）。它们的具体区别可见[https://www.msys2.org/docs/environments/](https://www.msys2.org/docs/environments/)。

另外，笔者曾经有使用MinGW的困难（[链接](https://kegalas.top/p/msys2%E4%B8%AD%E4%BD%BF%E7%94%A8mingw64%E7%9A%84g-%E7%BC%96%E8%AF%91%E8%BF%90%E8%A1%8C%E6%8A%A5%E9%94%99%E6%97%A0%E6%B3%95%E6%89%BE%E5%88%B0%E5%85%A5%E5%8F%A3/)），不过现在我觉得可能不是MinGW的问题，具体什么问题由于我现在没有再遇到，可能不能解决了。

注意，你安装ucrt64中的gcc后，可能会遇到编译过后，exe文件运行不正常的情况，例如打开直接错误退出，例如emacs的native compile错误的问题。这很可能是因为，你的电脑里面有太多g++的库了，比如GPG4Win，git，cmake，emacs，qt都会自带libstdc++-6.dll，这是gnu实现的c++的STL，这可能没什么，但是如果它们都在环境变量里，即使不会干扰到各自的运行（因为在同一个目录下的dll优先级高于在环境变量里的），也会对你自己编译的程序造成影响。解决办法是，把ucrt64的环境变量优先级拉高；或者复制一份ucrt64的libstdc++-6.dll到你的build目录下；或者干脆静态链接libstdc++。

如果你要打包发送给别人用，别人一般不会拥有libstdc++这个库，你要么选择静态链接，要么把dll一起打包发布。可能这里静态链接会有被GPL感染的风险（我查到的说法基本上各执一词），建议还是动态链接吧。
