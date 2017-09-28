---
title: 往beamer中添加高亮源代码
tags:
  - highlight
  - linux
  - tex
  - ubuntu
date: 2011-05-01 16:37:00
---

在ubuntu下使用highlight软件可以非常简单的高亮目标代码：

*   安装highlight：`sudo apt-get install highlight`
*   假如你有一个名叫`hello.c`的源文件，想要在一个tex文档中高亮显示它：
	1. `highlight -T -i hello.c -o hello.tex`
	2. 在`hello.c`所在路径下，将出现两个文件：`hello.tex`和`highlight.sty`
	3. 把`highlight.sty`拷贝到你的tex源文件所在路径，在你的tex源文件的导言部分(Preamble)加入：`\input {highlight.sty}`，并把`hello.tex`中所有内容全部插入到tex文档中需要显示高亮代码的位置。

以上就是全部的步骤，下面是效果图：
<div class="separator" style="clear: both; text-align: center;">[![](/img/frame_small.png)](/img/frame.png)</div><div>
</div>
