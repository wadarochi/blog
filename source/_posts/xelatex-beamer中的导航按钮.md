---
title: xelatex + beamer中的导航按钮
tags:
  - tex
date: 2011-05-01 13:39:00
---
# 缘由
在使用beamer制作Presentation的幻灯片页面中，导航按钮(官方名称是：navigation symbols)是非常重要的一个页面元素，在默认的情况下，每一页幻灯片都会有这些按钮：
<div class="separator" style="clear: both; text-align: center;">[![](/img/frame_page_small.png)](/img/frame_page.png)</div>
<div class="separator" style="clear: both; text-align: left;"></div>
<div class="separator" style="clear: both; text-align: left;">上图中各个按钮解释如下(摘自beamer user guide --&gt;&nbsp;8.2.4&nbsp;The Navigation Symbols)：</div>

1.  A slide icon，点击中间的方框将弹出一个对话框，询问你想要跳转到文档的第几页，点击方框的左右两边的箭头将跳转到前一页或者后一页幻灯片；
2.  A frame icon，点击左侧箭头将跳转到当前frame的第一页幻灯片，点击右侧箭头将跳转到当前frame的最后一页幻灯片，当同一个frame中有多页幻灯片的时候(比如使用了``\pause''之类的Overlay手段)，这连个按钮是很有用的；
3.  &nbsp;A subsection icon，点击左侧箭头将跳转到当前subsection的第一页幻灯片，点击右侧箭头将跳转到当前subsection的最后一页幻灯片；
4.  A section icon，点击左侧箭头将跳转到当前section的第一页幻灯片，点击右侧箭头将跳转到当前section的最后一页幻灯片；
5.  A presentation icon，点击左侧图标将跳转到第一页幻灯片，点击右侧图标将跳转到的最后一页(不包含附录)幻灯片；
6.  左侧类似撤销箭头一般的图标表示跳转到刚刚播放过的幻灯片页面，右侧按钮效果类似，点击中间查找模样的图标，将弹出对话框(或者在屏幕上方显示工具栏)，可以在其中输入关键词进行全文查找。

注意，在一些pdf浏览器中，这些按钮的部分或者全部功能是得不到支持的，比如sumatraPDF，所以在做Presentation的时候最好使用okular或者adobe reader。

正如标题中所暗示到的，使用xelatex编译的beamer的时候，这个导航按钮会失效，具体原因可以参考这个帖子：
<div style="text-align: center;">[[幻灯片] 解决beamer在XeLaTeX中导航条按钮失效问题](http://bbs.ctex.org/forum.php?mod=viewthread&tid=64104)</div><div style="text-align: left;"></div>

上文也提到了解决方案，在导言区(Preamble)加入如下代码即可：
``` latex
\makeatletter
\def\beamer@linkspace#1{%
	\begin{pgfpicture}{0pt}{-1.5pt}{#1}{5.5pt}
		\pgfsetfillopacity{0}
		\pgftext[x=0pt,y=-1.5pt]{.}
		\pgftext[x=#1,y=5.5pt]{.}
	\end{pgfpicture}}
\makeatother
```
