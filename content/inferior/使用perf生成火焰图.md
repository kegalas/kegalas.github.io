---
title: "使用perf生成火焰图"
date: 2025-03-01T19:07:37+08:00
draft: false
tags:
  - Linux
  - C++
categories: Linux
mathjax: true
markup: goldmark
---

对于性能优化的需求，我们经常会用到火焰图这个方法。Linux下确实有一种比较简单的生成方法，他需要用到perf来生成具体的运行统计信息，然后用[FlameGraph](https://github.com/brendangregg/FlameGraph)来将这个图画出来。

根据你的发行版，perf工具可能有不同的安装方式。例如在Ubuntu中可以通过

```bash
# WSL2 Ubuntu20.04
sudo apt install linux-tools-common
# 也有可能是linux-tools-generic
```

来安装。但是在WSL2的Ubuntu 20.04上，其可能会报错如下

```bash
WARNING: perf not found for kernel 5.15.153.1-microsoft
```

原因是perf这个工具和内核强相关，而WSL2的Ubuntu似乎魔改过内核，所以软件包中下载的perf用不了（相反，WSL2 Debian安装`linux-perf`就没这个问题，不知道为什么）。但我们还可以通过编译来安装。首先下载源码

```bash
# WSL2 Ubuntu20.04
sudo apt install linux-source
```

具体版本可能因机器而异，我这里装好的是linux-source-5.4.0，接着我们给它解压，编译，安装

```bash
# WSL2 Ubuntu20.04
sudo cp /usr/src/linux-source-5.4.0/linux-source-5.4.0.tar.bz2 ~
cd ~
tar -xf linux-source-5.4.0.tar.bz2
cd linux-source-5.4.0/tools/perf
make
make install
```

`make`的过程中可能会提示你缺少了`flex,bison`等工具，按照报错安装一下就行了。安装好之后perf的路径一般会是`~/bin/perf`，如果没有添加到PATH自行添加。接着我们下载FlameGraph

```bash
git clone https://github.com/brendangregg/FlameGraph.git
```

对于某一个程序`program`，我们通过以下形式生成其运行报告

```bash
perf record -g -F 99 -- ./program -i xxx.txt
```

这里前面的`-F`是采样频率99Hz的意思。后面的`-i xxx.txt`的意思是说明，你调用程序的命令行参数像原来一样直接加上就可以了，没参数就不加。

之后你运行`perf`的目录就会生成一个`perf.data`文件，然后

```bash
perf script -f | /path/to/FlameGraph/stackcollapse-perf.pl | /path/to/FlameGraph/flamegraph.pl > flame.svg
```

即可生成这个火焰图。

当然，以上内容仅linux可用，因为perf关系到linux内核，所以即使是msys2也没法做到。如果你在用windows，最好的方法可能就是使用visual studio提供的功能去分析。当然也有使用[gprof+graphivz](https://www.cnblogs.com/lidabo/p/15855580.html)的方法，我以前就是用这个，但是这个调用图不怎么好用，gprof我经常遇到不能生成`gmon.out`的问题，而且gprof似乎并不支持多线程，于是我放弃了，转向perf。
