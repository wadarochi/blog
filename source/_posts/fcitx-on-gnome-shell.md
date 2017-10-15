---
title: "GNOME Shell 3.24中的GTK程序也可以用fcitx啦"
date: 2017-10-15 02:21:30
tags:
 - gnome_shell
 - fcitx
---

# OS和Desktop Environment

操作系统：Ubuntu 17.04
桌面环境：GNOME Shell 3.24 on wayland

# 问题详情
Ubuntu 17.04可以安装gnome shell on wayland，得知这个消息之后我立即扔掉了默认的桌面环境，切换到了gnome shell on wayland。至今已经使用了几个月，大部分情况下还算挺稳定，不过也遇到了一些问题：

1. gnome-mplayer在wayland下无法工作，因为：it only works under X11
2. fcitx在所有的GTK和QT程序中都没法用，注意不是ctrl-space的问题，是没法用，但是在chromium中又可以正常使用
3. gnome-terminal如果开启透明背景，窗口四周的阴影无法征程渲染，会显示成一个相同宽度的白色边框

其实除了第2点，其它的都可以忍，今天要在笔记本上写中文blog，只好又去放狗搜索解决方案，看到的问题分析有以下几类：

## fcitx没有正常启动
这种情况下需要自己添加启动项，在gnome shell启动的时候去启动fcitx，否则chromium里面也不能用fcitx，但是我之前已经解决掉这个问题了（可以可以使用"GNOME 3 设置工具"来添加启动项）。

## 不能使用CTRL-SPACE来切换输入法

```bash
gsettings set org.gnome.settings-daemon.plugins.xsettings overrides "{'Gtk/IMModule':<'fcitx'>}"
```

不过我这里不是切换的问题，是没法使用，另外在chromium中，这个快捷键切换fcitx是有效的，看起来之前我已经敲了一遍了。

## 环境变量

```bash
# 向~/.xprofile中间添加如下内容：
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```

赶紧检查了一下，之前我已经添加过这3行内容了，但是`echo GTK_IM_MODULE`的结果依然为空，后来在arch linux的fcitx wiki上看到，对于新版本的gnome，需要把这三行加到`/etc/environment`这个文件中去......

添加，注销，重新登入，果然可以使用在gnome-terminal中使用fcitx了。
