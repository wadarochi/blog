---
title: xelatex + beamer中的字体设置
tags:
  - tex
date: 2011-05-01 15:30:00
---

在Windows 7下，我是这么设置的(以下代码写在导言区)：

<div class="highlight" style="background: #f8f8f8;"><pre style="line-height: 125%;"><span style="color: green; font-weight: bold;">\usepackage</span><span style="color: green;">{</span>xeCJK<span style="color: green;">}</span>
<span style="color: green; font-weight: bold;">\setCJKmainfont</span><span style="color: #7d9029;">[Mapping=tex-text]</span><span style="color: green;">{</span>Microsoft YaHei<span style="color: green;">}</span>
<span style="color: green; font-weight: bold;">\setCJKmonofont</span><span style="color: #7d9029;">[Mapping=tex-text]</span><span style="color: green;">{</span>SimSun<span style="color: green;">}</span>
<span style="color: green; font-weight: bold;">\setmainfont</span><span style="color: #7d9029;">[Mapping=tex-text]</span><span style="color: green;">{</span>TeX Gyre Pagella<span style="color: green;">}</span>
<span style="color: green; font-weight: bold;">\setmonofont</span><span style="color: #7d9029;">[Mapping=tex-text]</span><span style="color: green;">{</span>Courier New<span style="color: green;">}</span>
<span style="color: green; font-weight: bold;">\setsansfont</span><span style="color: #7d9029;">[Mapping=tex-text]</span><span style="color: green;">{</span>Trebuchet MS<span style="color: green;">}</span>
</pre></div>
需要说明的是：

*   假如你不喜欢这个字体配置或者你的操作系统里面么没有以上字体的话，改掉``{}''中的字体就可以了；
*   双引号的问题，xelatex编译beamer时不会将``''自动转换为“”，这个让人很不适应，所以使用``<span class="Apple-style-span" style="color: #7d9029; font-family: monospace; line-height: 16px; white-space: pre;">[Mapping=tex-text]</span>''改回来。再来说字体大小，xelatex编译beamer的时候，里面的一些元素的字体大小十分O疼，比如图片的标题(caption)字体大小默认很大(生怕后排的人看不见...)，如果你的页面需要混排公式和图片的话，这个就是一个灾难。

那么就手工改小吧，不过曾经在网上搜索到的一些方法已经失效了，没法偷懒找现成的方案就只能自己去翻《beamer user guide》，在``18.3.3&nbsp;Setting Beamer’s Fonts''部分有说明，大意是可以用``\setbeamerfont*{beamer-font name}{attributes}''来设置beamer中各种元素的字体属性。以下是我的例子(同样需要添加在导言区)：

<div class="highlight" style="background: #f8f8f8;"><pre style="line-height: 125%;"><span style="color: green; font-weight: bold;">\setbeamertemplate</span><span style="color: green;">{</span>caption<span style="color: green;">}</span>[numbered]
<span style="color: green; font-weight: bold;">\setbeamerfont</span><span style="color: green;">{</span>caption<span style="color: green;">}{</span>size=<span style="color: green; font-weight: bold;">\footnotesize</span><span style="color: green;">}</span>
<span style="color: green; font-weight: bold;">\setbeamerfont</span><span style="color: green;">{</span>captionname<span style="color: green;">}{</span>size=<span style="color: green; font-weight: bold;">\footnotesize</span><span style="color: green;">}</span>
</pre></div>

1.  第一行给图片标题加了编号；
2.  第二、三行把``caption''和``captionname''的字体大小改成了``<span class="Apple-style-span" style="color: green; font-family: monospace; font-weight: bold; line-height: 16px; white-space: pre;">\footnotesize</span>''；
3.  还可以改其它的字体属性，具体请参考用户手册。<div>不过这个时候就有一个大问题，我怎么知道beamer中到底有哪些元素？很遗憾，我在《beamer user guide》中没有找到相应的描述，难道要去翻源代码不成？好在我找到了一个网页：</div><div style="text-align: center;">[http://www.gphysics.net/latex/beamer.html](http://www.gphysics.net/latex/beamer.html)</div><div>里面列了一个很详细的表格，其中element那一项应该就是各种元素的名字，总之上面的``caption''和``captionname''都是这么找到的...</div>