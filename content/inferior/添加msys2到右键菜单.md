---
title: 添加msys2到右键菜单
date: 2024-01-15T22:40:43+08:00
draft: false
tags:
  - 计算机
  - MSYS2
  - Windows
  - 开发环境
  - 水文
categories: 其他计算机科学
markup: goldmark
---

参考自[https://gist.github.com/elieux/ef044468d067d68040c7](https://gist.github.com/elieux/ef044468d067d68040c7)

但是他那个时候还没有ucrt64，所以我略作修改如下。

首先是添加注册表项（保存成.reg文件运行即可）：

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

当然，我自己觉得这样比较丑。因为右键菜单是会按照字母顺序排序的，这样这四个就不在一起了。如果你想加二级菜单，那么可以如下进行

```reg
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\Directory\background\shell\Msys2SubMenu]
"MUIVerb"="Msys2"
"SubCommands"="Msys2Here; Ucrt64Here; Mingw64Here; Clang64Here"

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\CommandStore\shell\Msys2Here]
@="Msys2 Here"
"Icon"="G:\\Program_Files\\msys64\\msys2.ico"

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\CommandStore\shell\Msys2Here\command]
@="G:\\Program_Files\\msys64\\msys2.exe bash"

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\CommandStore\shell\Ucrt64Here]
@="Ucrt64 Here"
"Icon"="G:\\Program_Files\\msys64\\ucrt64.ico"

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\CommandStore\shell\Ucrt64Here\command]
@="G:\\Program_Files\\msys64\\ucrt64.exe bash"

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\CommandStore\shell\Mingw64Here]
@="Mingw64 Here"
"Icon"="G:\\Program_Files\\msys64\\mingw64.ico"

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\CommandStore\shell\Mingw64Here\command]
@="G:\\Program_Files\\msys64\\mingw64.exe bash"

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\CommandStore\shell\Clang64Here]
@="Clang64 Here"
"Icon"="G:\\Program_Files\\msys64\\clang64.ico"

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\CommandStore\shell\Clang64Here\command]
@="G:\\Program_Files\\msys64\\clang64.exe bash"

```

然后删除的代码就是

```reg
Windows Registry Editor Version 5.00

[-HKEY_CLASSES_ROOT\Directory\background\shell\Msys2SubMenu]
[-HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\CommandStore\shell\Msys2Here]
[-HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\CommandStore\shell\Ucrt64Here]
[-HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\CommandStore\shell\Mingw64Here]
[-HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\CommandStore\shell\Clang64Here]
```
