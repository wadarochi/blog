---
title: GNU screen encoding的替代品，自制BBS转码脚本
tags: []
date: 2011-05-24 22:36:00
---

之前一直想从screen迁移到tmux上来，因为后者相比较前者而言对脚本的支持更好，我可以轻松的将常用的几个窗口(window)打包成一个会话(session)，然后可以轻松的在一个脚本中将整个会话启动。但是一直下不定决心，因为screen有encoding功能，而tmux没有：
  > encoding enc [enc]
> 
> Tell&#160; screen&#160; how&#160; to&#160; interpret&#160; the&#160; input/output. The first argument sets the      
> encoding of the current window. Each window can emulate&#160; a&#160; different&#160; encoding.       
> The optional second parameter overwrites the encoding of the connected terminal.       
> It should never be needed as screen uses the locale setting to detect the encod‐       
> ing.&#160;&#160; There is also a way to select a terminal encoding depending on the termi‐       
> nal type by using the &quot;KJ&quot; termcap entry.
> 
> Supported encodings are eucJP, SJIS, eucKR, eucCN, Big5,&#160; GBK,&#160; KOI8-R,&#160; CP1251,      
> UTF-8,&#160;&#160; ISO8859-2,&#160;&#160; ISO8859-3,&#160; ISO8859-4,&#160; ISO8859-5,&#160; ISO8859-6,&#160; ISO8859-7,       
> ISO8859-8, ISO8859-9, ISO8859-10, ISO8859-15, jis.
> 
> See also &quot;defencoding&quot;, which changes the default setting of a new window.  

简言之，encoding可以让你为不同的窗口(window)设置不同的字符集编码(charset)，并根据你的设定自动进行编码转换(当然，并不仅限于此，后面会细说)，对于我们这些使用Linux服务器在天朝教育网挂各大BBS站的同学而言，这个功能是十分重要的，因为大陆的BBS都是GBK编码的(据我所知是这样的)，但是自己的Linux服务器的系统locale一般都是UTF-8，以前曾经写过一些screen后台挂站的文章：

1.  [[FAQ]控制台下登陆BBS](http://bbs.seu.edu.cn/bbsanc.php?p=22-4-10-5)2.  [CLI环境之BBS攻略](http://bbs.seu.edu.cn/bbsanc.php?p=22-4-10-23)  

有同学可能要说了，如果要转码的话，用luit不就行了？比如``luit –encoding GBK ssh –1 [guest@bbs.seu.edu.cn’’](mailto:guest@bbs.seu.edu.cn&rsquo;&rsquo;)，单纯从转码的角度来说，luit确实是足够强劲了，这里先岔开来介绍一下luit:
  > Luit&#160; is&#160; a&#160; filter that can be run between an arbitrary application and a UTF-8 terminal emulator.&#160; It will convert application output from the locale's&#160; encoding&#160; into&#160; UTF-8, and convert terminal input from UTF-8 into the locale's encoding.  

但是，luit在putty和gnome-terminal下面处理ASCII艺术(ASCII art or ANSI art)的时候是让人失望的，比如如下两幅截图：

[![View BBS through luit](http://lh6.ggpht.com/_5Si8_iQ2HPI/TdvCVkAA4BI/AAAAAAAAAGM/yTOMfOe-8Sc/luit_thumb%5B42%5D.png?imgmax=800 "View BBS through luit")](http://lh6.ggpht.com/_5Si8_iQ2HPI/TdvCU5a7bUI/AAAAAAAAAGI/JgMtEn4V8mk/s1600-h/luit%5B44%5D.png)[![View BBS through screen](http://lh3.ggpht.com/_5Si8_iQ2HPI/TdvCYNgzo1I/AAAAAAAAAGU/xPWu2J34j5E/screen_thumb%5B1%5D.png?imgmax=800 "View BBS through screen")](http://lh6.ggpht.com/_5Si8_iQ2HPI/TdvCXIwGLjI/AAAAAAAAAGQ/PSSG1d8OZvY/s1600-h/screen%5B3%5D.png)

左图是用luit登陆BBS时的惨状，右图是使用screen登陆BBS时的美景，真是对比出差距，不服不行啊……

其实我一开始也纳闷，按道理这两软件在对问题的处理上，都是先打开两个PTY，一个是对子程序的，以后是输出给用户的，然后自己通过在中间倒腾数据的时候做了相应的转码，怎么差距这么大呢……原来是screen在处理这些图形字符(line drawing characters)的时候，人为的给每个图形字符后面加了一个空格(有兴趣的同学可以翻翻screen的源代码，真相在encoding.c这个文件的recode_char_dw函数中；或者自己抓包看看)，所以图形变美观了。至于为什么要交空格，这个要涉及到GBK编码的字符宽度问题，本来这些图形字符在GBK中应该是和汉字等宽的，结果转UTF-8之后变得和英文字母等宽了，要是不补这个空格的话就会和luit一样乱套了，所以前面说screen干的不仅仅局限在狭义的转字符集编码，还把这个问题也考虑进去了。

好了，这么一说，screen太牛了，就算原生脚本支持不太好，用shell脚本(参见：[Run script that sends commands to the screen session it is being run in](http://stackoverflow.com/questions/899609/gnu-screen-run-script-that-sends-commands-to-the-screen-session-it-is-being-run))凑合一下也行啊，可惜screen在encoding上有一个bug，当你想设置encoding之后的窗口中输入图形字符(比如``※’’)的时候，这个符号会变成两个问号??，这可把我郁闷坏了，之前忍了好久了……其实这个bug在07年得时候就有人提交了，但是screen官方一直没动静，我翻了翻源代码，问题应该是处在encoding.c文件中的WinSwitchEncoding函数中，里面对字符宽度的计算出了问题，直接把这类图形字符变成了两个问号??，有兴趣的同学修改一下交给patch给开发者吧，功德无量啊~~

有了“变问号”和“原生脚本支持不足”这两点，已经足够让人又造轮子的打算了。不过我写这个工具之前还是用Google搜了一遍，确保没有重复的造轮子，用了如下关键词：``screen tmux encoding’’，``luit encoding tmux’’，``BBS ANSI tmux’’等等等等。

确实没有找到现成的轮子，然后我就用python写了这个工具(里面包含了pexpect_ng.py文件，从pexpect修改而来)，里面对图形字符的处理和screen一样，用正则表达式在每个图形字符之后加一个空格；名字现在叫做seu_bbs，但是其实把seu_bbs.py这个文件中第159行的``bbs.seu.edu.cn’’改成你喜欢的BBS域名比如``bbs.newsmth.org’’，用来登陆任何华语BBS应该是不成问题的。文件打包在这里：[seu_bbs.py 0.3版 &lt;挂站必备工具&gt;](http://bbs.seu.edu.cn/bbscon.php?bid=22&amp;id=41059)，里面也有一些介绍，这里不罗嗦了，贴一下。./seu_bbs.py -h的输出：
  > usage: seu_bbs.py [-h] [-a] [-e CHARSET] [-u USERNAME] [-t TIMEOUT] [-v]
> 
> convert charset of BBS to locale
> 
> optional arguments:     
> &#160; -h, --help&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; show this help message and exit      
> &#160; -a&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; whether or not seu_bbs.py take whole window.[default      
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; disable]      
> &#160; -e CHARSET, --encoding CHARSET      
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Set up seu_bbs.py to use CHARSET encoding.[default      
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; GB18030]      
> &#160; -u USERNAME, --username USERNAME      
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Set up seu_bbs.py to use USERNAME login bbs.[default      
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; guest]      
> &#160; -t TIMEOUT, --timeout TIMEOUT      
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Do someting when idle(unit is seconds, negtive value      
> &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; will be ignored).[default -1]      
> &#160; -v, –version  

这个工具目前还是有一些问题的，在从BBS的GBK转码为UTF-8的过程中，个别字符序列会转换不成功，我还没有详细的去调试到底是什么字符(估计是控制字符或者图形字符)导致转码失败，不过python内置的unicode函数倒是可以比较漂亮的忽略这个别转码错误，所以暂时就不管了，以后有时间再说吧。同时这个脚本的存在可以取代以往使用expect进行挂站，所以是完全的绿色版啊(除了需要python版本新于2.3以外)

最后要在赞一下pexpect模块的作者，代码的可读性太棒了！

btw:关于screen和tmux的比较，有两篇文章已经讲的很好了：

1.  [[LinuxTOY]从 screen 切换到 tmux](http://linuxtoy.org/archives/from-screen-to-tmux.html)2.  [Screen vs tmux](http://www.wikivs.com/wiki/Screen_vs_tmux)