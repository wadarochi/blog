---
title: 如何干净的修改ubuntu中gcc的版本
tags:
  - linux
date: 2010-05-03 02:34:00
---

这里要介绍的方法是update-alternatives[准确的说，应该是伟大的debian发行版的成员]。
man update-alternatives就会知道：
> update-alternatives - maintain symbolic links determining default commands要改大便系统的gcc版本，首先就需要把对应版本的gcc给装上[这个命令大家都知道吧]，假设原来系统的gcc版本是4.3，然后你新装了4.1的，那么输入如下命令:
> <pre>sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.3 40
> sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.1 30
> sudo update-alternatives --config gcc
> </pre>

现在，你需要的仅仅是按照提示将gcc默认指向的文件从4.3改为4.1即可。话说以前我都是直接改链接的，总觉得有点脏...