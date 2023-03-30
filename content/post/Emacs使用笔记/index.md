---
title: "Emacs使用笔记"
date: 2023-01-20T16:09:29+08:00
draft: false
tags: [Emacs,计算机]
categories: 其他计算机科学
mathjax: true
markup: pandoc
---

# 原版快捷键

## 光标、页面移动相关

|按键|功能|
|-|-|
|C-v|下一屏|
|M-v|上一屏|
|C-l|重绘屏幕，将光标至于中央、顶端、底端|
|C-p|方向键上|
|C-n|方向键下|
|C-b|方向键左|
|C-f|方向键右|
|M-f|这个词的末尾|
|M-b|这个词的开头|
|C-a|行首|
|C-e|行尾|
|M-a|这一句的开头|
|M-e|这一句的末尾|
|M-<|文件开头|
|M->|文件末尾|
|C-M-v|在另一个window里下一屏|
|C-M-S-v|在另一个window里上一屏|
|M-g M-g 行号|跳转到某一行|

## 文本操作

|按键|功能|
|-|-|
|\<DEL\>|删除(delete)光标前的一个字符，windows上为Backspace|
|C-d|删除光标后的一个字符|
|M-\<DEL\>|移除(kill)光标前的一个词|
|M-d|移除光标后的一个词|
|C-k|移除光标到行尾间的字符|
|M-k|移除光标到句尾间的字符|
|C-\<SPC\>|开始标记范围，从光标处开始，移动光标扩大选中范围|
|C-w|移除选中范围的字符，其实就是剪切|
|M-w|复制选中范围的字符|
|C-y|召回(yanking)被最近一次移除(kill)的字符|
|M-y|在C-y之后，紧接着使用，可以替换为召回再上一次被移除的字符，连续使用多次则替换为召回在上多次被移除的字符|
|C-/|撤回，多次使用则撤回更以前的操作。如果使用C-g打断undo，则之后的C-/是重做|
|C-s|向下文搜索。再次按C-s则搜索下一个出现的位置。\<Return\>即回车将光标停在这个位置，结束搜索。C-g结束搜索并将光标还原到原来的位置|
|C-r|向上文搜索,其他类似|
|M-x replace-string|替换字符串|
|C-o|与回车不同的是，光标不会进入到下一行|
|C-x C-o|将光标前后的所有空白行变成一个空白行|
|C-x h|全选|

## 文件相关

|按键|功能|
|-|-|
|C-x C-f|寻找并打开一个文件，如果不存在，则新建这个文件|
|C-x C-s|保存文件|
|C-x C-r|以只读方式打开文件|
|C-x C-q|切换为只读模式|

## Buffer相关

|按键|功能|
|-|-|
|C-x C-b|列出Buffer|
|C-x b 缓冲区名|切换到这个缓冲区|
|C-x s|保存多个缓冲区|

## Window相关

|按键|功能|
|--|--|
|C-x 0|关闭当前光标所在的window|
|C-x 1|保留当前光标所在的window，关闭其他所有|
|C-x 2|将屏幕分为上下两个window|
|C-x 3|将屏幕分为左右两个window|
|C-x o|光标切换到另一个window|
|C-x 4|以该命令为前缀时，表示在另外一个窗口做……例如C-x 4 f表示在另一个窗口打开新文件|

## 其他

|按键|功能|
|-|-|
|C-u 数字 其他命令|给其他命令传递一个数字，大部分都是重复次数|
|C-g|取消没输完的命令或者正在执行的命令|
|C-x|字符扩展，之后输入另一个字符或组合键|
|M-x|命令名扩展，之后输入一个命令名|
|C-x C-c|退出Emacs|
|C-x C-=|放大字号|
|C-x C\-\-|减小字号|
|C-x C-0|默认字号|

# 自用插件

## use-package

在init.el里自动安装其他插件的必备。通常可以用M-x package-install来手动安装use-package。

## good-scroll

平滑滚动的插件。

## mwim

优化了原版的C-a和C-e快捷键，具体如下。

|按键|功能|
|-|-|
|C-a|跳到文字的开头或这一行的开头|
|C-e|跳到文字的结尾或一行的结尾|

## counsel、ivy、swiper

三个著名的插件，优化了诸多功能，如搜索、切换buffer、文件操作、命令列表等等。

下面的命令可能会根据我使用的熟练程度来更新。

|按键|功能|
|-|-|
|C-s|使用swiper代替原版的搜索|
|C-x b|替代原版的buffer列表|
|C-x C-f|替代原版的文件操作|
|M-x|替代原版的命令列表|

## amx

将我们在M-x中输入命令的历史记录下来，每次显示最常用的。

## ace-window

|按键|功能|
|-|-|
|C-x o|优化切换window的操作逻辑，可以根据编号进行切换|

## undo-tree

提供比原版更好的撤销、重做操作。

|按键|功能|
|-|-|
|C-x u|打开undo tree|

## which-key

在输入快捷键时提醒我们可以接下来输入什么，以及有什么功能。

## flycheck

语法检查程序，对于c/c++需要装好clang才能使用。另外在windows上并不是完美支持的，虽然github的issue里最近没有什么东西。

## dashboard

一个欢迎界面。

## highlight-symbol

高亮Buffer中所有的、与光标处符号相同的符号。按<f3>开启。

|按键|功能|
|-|-|
|\<f3\>|开启高亮|

## rainbow-delimiters

彩虹括号，方便在lisp系语言中看清。

## company

自动补全插件。

|按键|功能|
|-|-|
|\<f1\>|显示候选项的文档（如果有、如果支持）|

## company-box

在图形界面下为company提供图标。以及可以开一个小的悬浮窗口展示候选项的文档（如果有）。但是其有一些问题，我觉得还是不开好。

## lsp-mode

代码分析。如定义跳转等等功能由lsp提供。

## lsp-ui

为lsp提供图形化的显示，同时

|按键|功能|
|-|-|
|M-.|寻找符号定义|
|M-?|寻找符号引用|

## lsp-ivy

使lsp和ivy协作，可以通过命令 lsp-ivy-workspace-symbol 来搜索当前工作区的符号。 

## projectile

项目管理插件。通常我们查找一个函数的定义或者别的什么的定义的时候，这些定义并不会在同一个文件里，而是在同一个项目里，此时我们lsp需要projectile才能正确查找。

## counsel-projectile

允许我们在项目中进行搜索。

## magit

内置git。

## neotree

打开文件夹树形结构图

|按键|功能|
|-|-|
|\<f8\>|打开neotree|

## all-the-icons

提供emacs内许多图标的显示功能，需要在下载插件后M-x all-the-icons-install-font。或是去其github下载fonts安装后才能使用。

## yasnippet

在补全的时候提供代码片段。

## avy

一套跳转光标的操作。暂时还不太会使用。

见[https://github.com/abo-abo/avy](https://github.com/abo-abo/avy)

## powerline

更好的mode-line显示。

## c++-mode

提供c++、c的支持。

## grip-mode

与markdown相关，还在探索中。

# 自用设置

## init.el

[https://github.com/kegalas/.emacs.d/blob/main/init.el](https://github.com/kegalas/.emacs.d/blob/main/init.el)

## 额外快捷键

|按键|功能|
|-|-|
|M-n|下十行|
|M-p|上十行|
|C-\<TAB\>|打4个空格，而不是像TAB在emacs中的智能缩进|
|C-c c|适用于打codeforces等竞赛，编译当前window里的单c++文件|

