---
title: 开发GNU Radio模块遇到的其它问题
tags:
  - gnuradio
  - linux
  - python
  - ubuntu
date: 2012-10-15 23:20:00
---

# 告别修改howto模块的历史
相信很多人在看完GNU Radio模块开发方面的文档之后，会出现“好麻烦啊”这样的想法。这也难怪，光是教学性质的**gr-howto-write-a-block**模块，就拥有7个子目录，108个文件。所以有一些同学是直接通过修改howto模块来开发自己的模块的，以此来减少一些体力劳动。

好在在[CGRAN](https://www.cgran.org/)上收集的项目中，我发现了这个方便的工具[devtools](https://www.cgran.org/wiki/devtools)：

> <span style="background-color: white; font-family: Arial, Verdana, Geneva, 'Bitstream Vera Sans', Helvetica, sans-serif; font-size: 14px; line-height: 19.03333282470703px; widows: 4;">This project aims to provide tools, helper, scripts and scriplets to aid the development of GNU Radio.</span>

简而言之，devtools通过提供名为modtool.py的工具，为各位开发者提供了一个快速生成模块骨架的手段。关于这个工具的简要介绍，可以参考[这里](https://www.cgran.org/wiki/devtools)，余下部分将简要介绍如何生成一个source模块的。

## 安装

1.  想要获得最新版本的话，请从其[项目主页](https://github.com/mbant/gr-modtool)(github)上下载最新版本的[gr_modtool.py](https://github.com/mbant/gr-modtool/blob/master/gr_modtool.py "gr_modtool.py")文件即可。注意，只需要这一个python文件即可；
2.  把[gr_modtool.py](https://github.com/mbant/gr-modtool/blob/master/gr_modtool.py "gr_modtool.py")文件扔到PATH中的一个路径即可，如此既安装完毕。

## 使用工具自动生成模块框架

1.  切换到合适的路径，比如~/Workspace/
2.  生成(Module)，假如你决定将自己的Module命名为PT，那么运行`gr_modtool.py create PT`之后，可以在当前路径下看到子目录`gr-PT`，结构如图一所示；
3.  为Module添加Block(信号处理的模块)，这一步才是最关键的步骤。如前文所述，我们假设现在要添加一个信号源模块，于是运行`gr_modtool.py add -d gr-PT/ -n PT -N simple_source -t source`，其中`add`表示向PT module中添加block，而`PT`则是module名，`simple_source`表示需要添加的block名，`-t source`表示添加的block类型是一个源；
4.  回车之后会要求输入参数列表(argument list)，也就是block构造函数的参数列表，在这里我们输入`int frequency`。其实输入参数带默认值也是可以的，比如`int val1, double val2=0`；
5.  之后还会询问是否需要生成Python或者C++的qa文件，一路回车即可；
6.  如此便大功告成，比如在`lib`路径下会多出`PT_simple_source.cc`和`qa_PT_simple_source.cc`两个文件，而其它的相关文件也会被悉数添加。不仅如此，最贴心的功能莫过于各种源文件中需要修改的地方都加了Vim的place holder，如图二所示。

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;"><tbody><tr><td style="text-align: center;">[![PT module directory tree structure](/img/new_module_structure.jpg "图一 新生成Module结构")](/img/new_module_structure.jpg)</td></tr><tr><td class="tr-caption" style="text-align: center;">图一 PT module文件结构</td></tr></tbody></table><table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;"><tbody><tr><td style="text-align: center;">[![](/img/place_holder.jpg)](/img/place_holder.jpg)</td></tr><tr><td class="tr-caption" style="text-align: center;">图二 贴心的place holder</td></tr></tbody></table><div>如果想要获取更详细的介绍，可以到[devtools](https://www.cgran.org/wiki/devtools)这里查看里面的例子，或者`modtool.py help add`或者RTFC。

## 编译模块可能遇到的问题
之前在编译gr-chancoding模块的时候，还遇到过一个比较常见的问题，./configure时：

> checking Python.h usability
> result: no
> checking Python.h presence
> result: no
> checking for Python.h
> result: no
> error: cannot find usable Python headers

但是诡异的是Python.h平躺在/usr/include/python2.7/下，并且configure脚本已经找到了这个路径。于是翻了下config.log，终于发现问题是：

> fatal error: asm/errno.h: No such file or directory

好吧，谁叫我没更新Ubuntu，又遇到这个老朋友了[Bug 48879 - Compilation cannot find file asm/errno.h](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=48879)。之后只管`./configure CFLAGS="-I /usr/include/i386-linux-gnu/"`即可。

## 第三方module加载失败
在make, make install都顺利通过之后，决定打开ipython手工加载chancoding模块试试，结果在执行`import chancoding`时获得如下错误信息：

> libgnuradio-chancoding.so.0: cannot open shared object file: No such file or directory

难道Python找不到`/usr/local/lib`这个路径吗...因为之前`./configure`的时候没有指定`--prefix`，那么module的安装路径就应该是`/usr/local/`当中，解决方法就是：

> export LD_LIBRARY_PATH="/usr/local/lib"这一行加到`~/.bashrc `当中即可。
