---
title: Linux开发环境配置合集（长期更新）
date: 2023-10-13T19:06:00+08:00
draft: true
tags:
  - 计算机
  - Linux
  - 开发环境
  - 水文
categories: Linux
markup: goldmark
---

# 系统安装

我一般安装Debian，安装的时候选择Cinnamon桌面环境，语言选择英文（方便tty等地方显示英文而不是方块字乱码，之后可以调整部分界面变成中文）。但是地区记得选中国上海。并且不知道为什么，虽然我选的是美式键盘，他还是帮我装上了fcitx5.

具体的安装，每次都有不同，就不介绍了。安装包一般可以在[https://mirrors.tuna.tsinghua.edu.cn/](https://mirrors.tuna.tsinghua.edu.cn/)获取。

另外，Debian一般只能下到stable版本的镜像文件，更新到testing或者sid只用更换软件源的版本即可。

# 系统配置

## 软件源

[https://mirrors.tuna.tsinghua.edu.cn/help/debian/](https://mirrors.tuna.tsinghua.edu.cn/help/debian/)里说的很清楚，只不过我这里偶尔会出现连不上的情况，字符搜索替换成阿里云的可以解决问题。

## 安装VIM

这个很重要，因为我们修改很多文件都要用vim而不是系统自带的精简版vim。

```bash
$ sudo apt install vim
```

## 设置中文locales

```bash
$ sudo vim /etc/locale.gen
```

在里面选择

    en_US.UTF8 UTF8
    zh_CN GB2312
    zh_CN.GBK GBK
    zh_CN.UTF8 UTF8

即把它们前面的注释取消掉。记得灵活使用vim的搜索功能。之后

```bash
$ sudo locale-gen
```

之后

```bash
$ sudo dpkg-reconfigure locales
```

可以看到这是一个图形化的选择locale的工具，但是他没有GB2312和GBK，所以我们还是不能用它选择。注意空格是勾选选项，回车是OK。我们直接OK，然后选择default为`zh_CN.UTF8 UTF8`。然后注销再登陆。

系统可能会提示你迁移`~`中的文件夹到新名称，不要迁移，因为我觉得英文目录会少很多麻烦。

不过我们这样非常粗暴的把全部locale都设置成中文是有问题的，在tty中，任何中文相关的东西都会变成方块、菱形等。我并没有找到什么单独设置每个地方的locale的简单办法，但是我发现我们可以临时改变locale。于是我们只要使用shell判断我们是在tty还是图形界面，来设置即可。

当我们用bash的时候，我们可以修改`~/.bashrc`，如果用zsh、fish等同理。我在bash后面添加了这一段代码

```bash
if [ "$XDG_SESSION_TYPE" = "tty"]; then
    export LC_ALL=en_US.UTF-8
    export LANG=en_US.UTF-8
    export LANGUAGE=en_US.UTF-8
fi
```

其中`if`的条件是最关键的地方，我的系统可以用`$XDG_SESSION_TYPE`来简单判断，别的系统你需要自己找。注意shell里面的空格没打对可能会影响句意。

另外，要在tty里root用户使用这个的话，还要加到`/root/.bashrc`里。

## 自动挂载

Windows会自动挂载硬盘，但是Linux不一定会。尤其是我们使用双系统时，如果部分盘没有挂在linux的目录，就不会自动挂载。我们要手动挂载，但是要输入密码，比较繁琐。如果我们想要避免输入密码，就可以去修改`/etc/fstab`

我们首先把所有盘都先挂在好，然后输入

```bash
$ sudo blkid
```

得到一大串输出，例如

```
/dev/sda2: LABEL="Games" BLOCK_SIZE=xxx UUID=xxx TYPE="ntfs"
```

我确实有一个盘叫Games，并且是ntfs格式的，就把他的UUID记下来，在`/etc/fstab`里新开一行

```fstab
UUID=xxx    /media/kegalas/Games    ntfs    defaults     0 0
```

UUID就是刚刚复制的那个，挂载路径看你Linux图形界面默认会挂在到哪里，就挂到相同的地方，我这里就是`/media/kegalas/Games`，`ntfs`看你的盘的格式，一般来说双系统Windows的盘都是ntfs的。后面不管直接输入default和0 0.

具体fstab每一项参数的含义可以上网查阅，或者`man fstab`

重启检查是否有效。

## 自动同步时间

很奇怪，我安装后Cinnamon的设置里并不能直接使用网络时间。我进行下面几个操作后才能正常使用

```bash
$ sudo apt install npd
$ sudo systemctl start ntpd
$ sudo systemctl enable ntpd
```

## 与Windows的八小时时差问题

```bash
$ sudo timedatectl set-local-rtc 1 --adjust-system-clock
```

在linux设置即可，但是系统也会提示你，这种方法可能会有一些问题。

# 开发环境配置

TODO

# 其他软件安装

可见[个人常用软件列表](../个人常用软件列表)一文。
