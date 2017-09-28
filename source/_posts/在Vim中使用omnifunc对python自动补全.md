---
title: 在Vim中使用omnifunc对python自动补全
tags: []
date: 2011-05-12 09:00:00
---

如果希望在使用Vim编辑python源码的时候能够获得自动补全功能，你并不需要安装第三方插件，从Vim 7开始，自带的'omnifunc'既可以提供这一利器。步骤：

1.  **确认你的Vim编译的时候包含了 `+eval`，`+insert_expand`，`python`**
当然如何确定自己机器上的Vim已经在编译时打开这三个选项了呢？在命令行模式输入`:version`后回车即可。但是对于`+python`的确定要稍微麻烦一些，光看到`+python`还不够，应该在命令行中执行一条python语句确认一下`:py print "hentai"`，假如执行之后没有报错，没有崩溃，并且最下一栏显示了`hentai`，那么恭喜你，`+python`特性是没有问题的；如果报错、崩溃(从Vim中退出之类的)，那么可能你遇到了如下的问题(这种情况下你很可能是用的Windows，用有包管理工具的Linux很难遇到这类问题)：
a.  python版本不兼容，这种情况是`+python/dyn`引发的，也就是说Vim中对python特性的支持是靠动态链接系统中python动态链接库来实现的，比如我装的Vim 7.3要求`python 2.7.dll`。不想重编译Vim的话就换匹配的python吧(这个可以在执行`:py print "hentai"`时看到提示的)
b.  假如你用的是64bit的Windows系统，然后用了32bit的Vim，再用了64bit的python，肯定会出问题，解决的方法比较简单把Vim换成64bit版本吧
c.  Vim编译进了`+python/dyn`和`+python3/dyn`两个选项也可能(**注意：这里是可能，我只是搜索的时候看到有人提过**)导致问题(官方的安装包就是这样的)，重编译Vim，关掉一个选项(关掉哪个要看你系统没装哪个)

2.  **在.vimrc(用Linux的话)或者_vimrc(用Windows的话)中加入以下设置**：

<div><span class="Apple-style-span" style="color: white; font-family: monospace;"></span>
<pre style="background-color: #333333; color: white; font-family: monospace;"><span class="Comment" style="color: skyblue;">""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""</span>
<span class="Comment" style="color: skyblue;">" For editing python</span>
<span class="Comment" style="color: skyblue;">""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""</span>
<span class="Statement" style="color: khaki; font-weight: bold;">filetype</span> <span class="Type" style="color: darkkhaki; font-weight: bold;">plugin</span> <span class="Type" style="color: darkkhaki; font-weight: bold;">indent</span> <span class="Type" style="color: darkkhaki; font-weight: bold;">on</span>
<span class="Statement" style="color: khaki; font-weight: bold;">set</span> <span class="PreProc" style="color: indianred;">completeopt</span>+=longest
<span class="Statement" style="color: khaki; font-weight: bold;">set</span> <span class="PreProc" style="color: indianred;">completeopt</span>+=menu
<span class="Statement" style="color: khaki; font-weight: bold;">set</span> <span class="PreProc" style="color: indianred;">wildmenu</span>
<span class="Statement" style="color: khaki; font-weight: bold;">autocmd</span> <span class="Type" style="color: darkkhaki; font-weight: bold;">FileType</span> python <span class="Statement" style="color: khaki; font-weight: bold;">set</span> <span class="PreProc" style="color: indianred;">omnifunc</span>=pythoncomplete#Complete</pre></div><div>> 如果发现补全完毕之后提示窗口没有自动关闭，就加上下面这两行(从stackoverflow上看到的)：
<div style="background-color: #333333; color: white; font-family: monospace;"><pre style="background-color: #333333; color: white; font-family: monospace;"><span class="Comment" style="color: skyblue;">" If you prefer the Omni-Completion tip window to close when a selection is</span>
<span class="Comment" style="color: skyblue;">" made, these lines close it on movement in insert mode or when leaving</span>
<span class="Comment" style="color: skyblue;">" insert mode</span>
<span class="Statement" style="color: khaki; font-weight: bold;">autocmd</span> <span class="Type" style="color: darkkhaki; font-weight: bold;">CursorMovedI</span> * <span class="Statement" style="color: khaki; font-weight: bold;">if</span> <span class="Identifier" style="color: palegreen;">pumvisible</span><span class="Special" style="color: navajowhite;">()</span> <span class="Statement" style="color: khaki; font-weight: bold;">==</span> <span class="Constant" style="color: #ffa0a0;">0</span>|<span class="Statement" style="color: khaki; font-weight: bold;">pclose</span>|<span class="Statement" style="color: khaki; font-weight: bold;">endif</span>
<span class="Statement" style="color: khaki; font-weight: bold;">autocmd</span> <span class="Type" style="color: darkkhaki; font-weight: bold;">InsertLeave</span> * <span class="Statement" style="color: khaki; font-weight: bold;">if</span> <span class="Identifier" style="color: palegreen;">pumvisible</span><span class="Special" style="color: navajowhite;">()</span> <span class="Statement" style="color: khaki; font-weight: bold;">==</span> <span class="Constant" style="color: #ffa0a0;">0</span>|<span class="Statement" style="color: khaki; font-weight: bold;">pclose</span>|<span class="Statement" style="color: khaki; font-weight: bold;">endif</span>
</pre></div></div>到这里就大功告成了，补全的按键组合是`Ctrl-X Ctrl-O`，只要系统中安装好的python库都能补全，效果如下(左侧边栏靠的taglist实现)：
<div class="separator" style="clear: both; text-align: center;">[![](/img/omni_small.png)](/img/omni.png)</div>
