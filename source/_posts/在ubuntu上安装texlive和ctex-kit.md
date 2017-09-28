---
title: 在ubuntu上安装texlive和ctex-kit
tags:
  - tex
  - ubuntu
date: 2011-10-06 23:33:00
---

texlive与ctex-kit的搭配，应该是我接触过的最简单、优雅的Linux TeX中文方案，水木TeX版置底推荐的也是这个。

但是以前在Ubuntu下安装的过程依然有些不上路子，因为诸如Debian、Ubuntu在内的各大Linux发行版软件仓库中，texlive的版本一度十分古老，所以假如想很顺利的用上texlive+ctex-kit的话，大家不得不自己下载texlive的镜像来安装。这样做的一个最直接的问题就是当你需要安装诸如apt源当中的vim-latexsuite的时候，你需要手工构建一个空包，欺骗包管理工具，让它以为你已经从apt源里面安装了texlive，这个方法多少有些快糙猛……

即使到了今天，我尝试过Ubuntu 11.10 beta 2之后，事情依然没有改观，Ubuntu官方的文档上时这么写的：
> _Note_: As of July 2011 the <tt>texlive</tt> package that ships with Ubuntu (TeX Live 2009) is [<u><span style="color: #0066cc;">lagging two years behind the current TeX Live release</span></u>](https://bugs.launchpad.net/ubuntu/+source/texlive-base/+bug/712521) (TeX Live 2011). If you want the latest version of TeX Live, you can install it directly from [<u><span style="color: #0066cc;">the TeX Live website</span></u>](http://www.tug.org/texlive/) (this does not interfere with the packages in Ubuntu).所以我还是选择了从iso直接安装，以下是安装步骤：

#    <span style="font-size: large;">0.准备工作</span>
首先，为了能使用texlive自带的聊胜于无的图形安装界面，Ubuntu用户需要安装perl-tk这个软件包(当然，喜欢文本安装的同学可以跳过这一步)：
> sudo apt-get install perl-tk

#    <span style="font-size: large;">1.下载texlive 2011 DVD iso</span>
对于大陆教育网的用户来说，ipv6网络真是好东西，不仅免费，速度还一级棒，兼顾自动翻墙，texlive的镜像也可以从中科大的镜像下载：
[http://mirrors6.ustc.edu.cn/CTAN/systems/texlive/Images/texlive2011.iso](http://mirrors6.ustc.edu.cn/CTAN/systems/texlive/Images/texlive2011.iso)

#    <span style="font-size: large;">2.挂载iso</span>
``` bash
cd /home/foo/bar> mkdir CDROM> sudo mount -t iso9660 /path/to/your/texlive2011.iso /home/foo/bar/CDROM
```

#    <span style="font-size: large;">3.安装texlive 2011</span>
``` bash
sudo -i cd /home/foo/bar/CDROM> ./install-tl --gui=wizard
```
然后，接下来的事情就比较简单了，一路下一步，直到`完成`。

#    <span style="font-size: large;">4.设置PATH环境变量</span>
因为texlive 2011的可执行程序默认是安装在/usr/local/texlive/2011/bin/i386-linux下的，所以需要修改path环境变量才能直接在shell中调用。
``` bash
export PATH=$PATH:/usr/local/texlive/2011/bin/i386-linux
```
这一行最好是加到你的`~/.bashrc`中去，以后省得麻烦。

#    <span style="font-size: large;">5.下载&amp;安装ctex-kit</span>
``` bash
mkdir -p ~/texmf/tex/latex
cd ~/texmf/tex/latex
svn co http://ctex-kit.googlecode.com/svn/trunk/ctex
```
自此，ctex-kit就装好了，非常简单。

#    <span style="font-size: large;">6.安装Adobe字体(原始链接：[这里](http://www.newsmth.net/bbscon.php?bid=460&amp;id=299823))</span>
从这里找到字体下载的链接：
> [http://bbs.ctex.org/viewthread.php?tid=47618](http://bbs.ctex.org/viewthread.php?tid=47618)

它们分别是：
*   AdobeFangsongStd-Regular.otf
*   AdobeHeitiStd-Regular.otf
*   AdobeKaitiStd-Regular.otf
*   AdobeSongStd-Light.otf

在用户主目录下建立如下子目录：
``` bash
~/.fonts/adobefonts/
```

将Adobe的四款字体解压到该子目录，安装字体：      
``` bash
cd ~/.fonts/adobefonts/       
sudo mkfontscale       
sudo mkfontdir       
sudo fc-cache -fv
```

安装完成后可以用      
``` bash
fc-list :lang=zh|grep -v 'Adobe'
```
检查一下

#    <span style="font-size: large;">7.用一个简单的例子试一下吧(原始链接：[这里](http://www.newsmth.net/bbscon.php?bid=460&amp;id=299823))</span>
``` tex
\documentclass[adobefonts]{ctexart}      
\begin{document}       
你好，TeX Live 2009！       
\end{document}
```

在命令行下用 xelatex test.tex 进行编译，如果顺利的话，就得到了pdf文件。

Note:

1.  Adobe的字体部分请各位三思，我先是用Google搜了半天，然后有到Adobe官网翻了一遍，也没搜到上面这四款字体是收费还是免费的，担心版权问题的同学还是老老实实用文泉驿字体，贴在这里是因为ctex-kit中刚好有ctex-xecjk-adobefonts.def，觉得可能是免费使用的。
2.  如果需要在Linux下面使用gvim + latex-suite的同学，可以参考[这篇文章](http://bbs.seu.edu.cn/bbscon.php?bid=230&amp;id=3209)。但是因为是去年写的，现在的texrc以及compiler都有少许不同了。比如前向搜索，现在okular和synctex之间的衔接除了点问题：[Synctex Forward-search is not working (Kile + Okular)](https://bugs.archlinux.org/task/25125) ，里面提到的可能的解决方案我也试了，暂时无解，只有等上游的okular更新了；反向搜索还是没有问题的。
