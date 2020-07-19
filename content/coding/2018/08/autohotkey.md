---
title: "用 Autohotkey 保持窗口在最前"
date: 2018-08-22T00:44:47+08:00
author: Alvin
keywords: autohotkey,always on top,窗口,最前
---

## AutoHotKey

在 linux 系统上，右键点击窗口的标题栏，选择 `Always on top`，就可以将窗口保持在最前，非常方便。windows 系统不带这个功能，可以用 [AutoHotkey](https://www.autohotkey.com/) 实现。

AutoHotkey 是款非常小巧的软件，安装后，在系统右键新建菜单中会出现 `AutoHotkey Script` 项，点击这个菜单，新建一个名为 always_on_top.ahk 的脚本文件，加上下面一行：
```cpp
^1::  Winset, Alwaysontop, , A
```

其中 `^1::` 表示绑定 `ctrl+1` 快捷键，可以换其他快捷键，官方文档中有详细说明。

双击运行上面的脚本文件，系统右下角会出现一个绿色背景的 `H` 图标。现在可以按 `ctrl+1` 将当前窗口保持在最前了，再按一次取消，还算方便。

## 开机启动

如果不想每次手动执行上面的脚本，可以将它加到开机启动项里面。在注册表中找到下面的位置：
```sh
[HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run]
```
新建一项就可以了。

如果实在是懒，可以新建一个 `xx.reg` 文件，内容如下：
```sh
Windows Registry Editor Version 5.00
[HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run]
"AlwaysOnTop"="D:\\path\\to\\always_on_top.ahk"
```

注意脚本的路径要填正确，然后双击 xx.reg 写到注册表里就 ok 了。
