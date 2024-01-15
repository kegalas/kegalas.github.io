---
title: 干货 | 为Hugo安装goldmark-mathjax插件来更好地支持输入公式
date: 2023-09-21T21:32:37+08:00
draft: false
tags:
  - Hugo
  - Goldmark
  - Mathjax
  - 干货
categories: 其他计算机科学
---

# 动机

在很久以前的使用中，我发现默认的Hugo渲染引擎无法正确渲染所有数学公式，但我并没有深究为什么。我进行一番搜索后改用pandoc来作为渲染引擎，它在数学公式方面表现的不错，但是，它无法生成目录，也无法生成代码高亮。

再一番搜索后，我搜索到这篇文章<u>[给 Hugo 增加一些 Pandoc 支持增强](https://bigshans.github.io/post/hugo-patch-with-pandoc/)</u>，里面详细介绍了怎么才能让pandoc正确生成目录，之后我写了文章<u>[重新编译hugo来使得pandoc可以生成目录](../干货-重新编译hugo来使得pandoc可以生成目录)</u>。

在这些操作后，目录确实可以生成了，但是代码高亮却没有出现。另外，这个版本比较老，无法使用Hugo的最新特性。

不过我暂时还是保持使用这个版本，因为比起高亮，正确显示公式才是最重要的。

现在我开始思考要怎么才能显示高亮。Hugo官方对于其他渲染引擎的支持很少（见[https://gohugo.io/content-management/formats/](https://gohugo.io/content-management/formats/)），例如pandoc，go-org，rst，asciidoc等。当然这里面asciidoc是个例外，不过我尝试了一下发现仍然不能生成目录。我最终转向了去研究怎么才能让默认的goldmark正确显示公式。

# 问题分析

我仔细观察了一下没渲染出公式的代码变成了什么样。最终，我发现，如果一个公式里面有两个下划线，就有可能渲染不成功，并且不成功的地方会变成斜体。再研究发现，两个下划线中间包含字符，是markdown本来的斜体语法（虽然我更常用两个\*号），这与Latex里面的下划线用法冲突了，并最终导致了这个问题。

# 解决办法

也有一些教程，例如[https://bwaycer.github.io/hugo_tutorial.hugo/tutorials/mathjax/](https://bwaycer.github.io/hugo_tutorial.hugo/tutorials/mathjax/)通过往html里面嵌入mathjax的js代码来解决问题，但是我自己试了一下好像并没有成功。

我搜索到了goldmark有mathjax插件，机缘巧合之下我又看到了这篇文章[Hugo&Eureka&Nginx](https://yearn.xyz/posts/tools/hugoeurekanginx/#%E6%B7%BB%E5%8A%A0-mathjax-%E6%94%AF%E6%8C%81)，明白为Hugo加入goldmark-mathjax是可能的，试验之后确实如此，实际操作过程如下。

## 环境需求

- Go 1.19或更新，GCC

官网没有提到GCC的版本，我这里使用的是msys2 ucrt里安装的gcc 13.1.0，而go用的是1.21.1。

## Clone、修改代码

```bash
git clone https://github.com/gohugoio/hugo.git
```

（不知道为什么shell代码没有高亮）

可能会遇到网络问题，需要自行解决。

之后在`hugo/markup/goldmark/convert.go`中进行修改。

- 在`import`部分添加`"github.com/litao91/goldmark-mathjax"`
- 在`extensions = append(extensions, images.New(cfg.Parser.WrapStandAloneImageWithinParagraph))`后添加`extensions = append(extensions, mathjax.MathJax)`

至此修改结束，可以看[这个链接](https://github.com/kegalas/hugo/commit/3e6847c5fdad23bc1beb24e05eb4b194c511f200#diff-f0561d87ec12103eaef3aa1b9e71eaffd3e86c4b42457cbdccdc92de96ebeed9)查看具体怎么修改的。也可以直接克隆我的fork，[https://github.com/kegalas/hugo](https://github.com/kegalas/hugo)

## 编译

```bash
cd hugo
go get github.com/litao91/goldmark-mathjax
go install --tags extended
```

后两个命令都需要网络通畅。

编译成功后，一般会生成在`C:\Users\XXX\go\bin`，找不到的建议可以用Everything搜一下hugo.exe。之后就可以愉快的使用了。

# 已知问题

1. 输入公式时，`<`有时要求后有空格，否则会渲染失败。一般来说，应该是`<`后加上大小写字母会渲染失败，必须加上空格。可以用搜索软件搜索所有笔记手动修改。
2. 两个行内公式，如果中间没有间隔，或者中间间隔为英文逗号、句号，会渲染失败。例如`$abc$$abc$`，`$abc$,$abc$`，`$abc$.$abc$`。后面两种情况搜索即可。第一种情况，由于正常的行间公式双`$`符在单独一行，所以直接搜索`$$`，obsidian只会把搜索结果的那一行显示，所以即使有几千条也还是比较方便找到的。
3. 行间公式，双`$`符要单独在一行，否则会渲染失败。解决方法同上。
