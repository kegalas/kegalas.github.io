---
title: 干货 | 重新编译hugo来使得pandoc可以生成目录
date: 2022-08-06T16:41:53+08:00
draft: false
tags:
  - Hugo
  - Pandoc
  - 干货
categories: 其他计算机科学
---

> 注：本文内容可能不再推荐使用，因为pandoc无法生成正确的代码高亮，也没有更多的参数可以选择。并且添加目录的方式比较DIY，而我本人暂时不会Go语言相关的东西，无法进一步跟进官方版本的更新。
> 
> 现在推荐去给Hugo默认的goldmark引擎添加mathjax支持。
> 
> 详情见这篇文章[为Hugo安装goldmark-mathjax插件来更好地支持输入公式](../干货-为hugo安装goldmark-mathjax插件来更好地支持输入公式)

根据[https://github.com/olOwOlo/hugo-theme-even/issues/139](https://github.com/olOwOlo/hugo-theme-even/issues/139)，hugo无法正确生成目录的原因是，没有加入--toc参数。

其中@jdhao所说的[https://github.com/gohugoio/hugo/blob/master/helpers/content.go#L735](https://github.com/gohugoio/hugo/blob/master/helpers/content.go#L735)这一部分代码现在（编辑日期时）已经转移到了[https://github.com/gohugoio/hugo/blob/master/markup/pandoc/convert.go#L67](https://github.com/gohugoio/hugo/blob/master/markup/pandoc/convert.go#L67)。按他的办法来说应该改成

```go
	args := []string{"--mathjax","--toc"}
```

实际上并没有那么简单，即使是加入了--toc参数也不能生成目录。笔者又找到一份他人修改的版本[https://github.com/bigshans/hugo](https://github.com/bigshans/hugo)。这个版本的convert.go明显是修改过的，能够正确生成目录。

将其克隆、编译，然后替换掉原来的hugo.exe。经测试可以正常使用。目前发现的唯一可以称得上是一个问题的是，代码没有高亮。

编译方法见hugo的github readme。

如果因为网络问题无法编译，可以给powershell设置代理，见之前的文章。

如果cgo exit status 2，那么可能是g++的问题，笔者的电脑上g++是msys2滚动更新的，更换为8.1.0版本成功编译了。（后注：要区分msys环境和mingw64环境，见这篇文章<u>[MSYS2,MinGW64,Cygwin的使用区别浅谈](../干货-msys2mingw64cygwin的使用区别浅谈)</u>）

