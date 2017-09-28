---
title: 自动安装gnuradio的脚本
tags:
  - gnuradio
date: 2011-05-15 10:36:00
---

首先埋汰一下Ubuntu自带的gnuradio版本，就算是在最新的测试版11.10上，gnuradio的版本也还是3.2.0，太旧了，以至于无法兼容USRP上的WBX之类的射频子板，运行时会出现如下错误：
> Warning: Treating daughterboard with invalid EEPROM contents as if it
> were a "Basic Tx."
> Warning: This is almost certainly wrong...  Use appropriate
> burn-*-eeprom utility.出现这个错误的根源是因为gnuradio 3.2.0版本中射频子板数据库中并没有WBX子板的ID(这个值是存储在各块子板的eeprom中)，解决的方法是更新到3.3.0版，或者直接从git源中安装gnuradio和uhd。通过Google，找到了一个安装脚本：
> [Build script for UHD+GnuRadio on Fedora and Ubuntu](http://www.sbrac.org/files/build-gnuradio)感谢原作者Marcus Leech，不过这个脚本存在一些问题，里面在对很多重要命令的执行结果进行判断的时候使用的是“grep前一个命令的输出”这样的方式，所以当这个脚本运行在非英语locale的系统上时，会出现相当多的误判。我把这些检查方式换成了判断前一个命令结果的返回值，应该能够适用于各种locale设置的Fedora、Ubuntu系统，修改后的版本在这里：[gnuradio_build](https://docs.google.com/leaf?id=0B3G77VEE7JciNWZmMGQyZjAtNjY5ZS00YTFjLWExNTItYjJlNWFmNjAzNmFj&amp;hl=zh_CN)(放在Google doc上了)，改动如下：

<div style="background-color: #333333; color: white; font-family: monospace;"><pre style="background-color: #333333; color: white; font-family: monospace;"><span class="Statement" style="color: khaki; font-weight: bold;">53,54c53,54</span>
<span class="Special" style="color: navajowhite;">&lt;     which $1 &gt;tmp$$ 2&gt;&amp;1</span>
<span class="Special" style="color: navajowhite;">&lt;     if grep -q no.${1}.in tmp$$</span>
<span class="Statement" style="color: khaki; font-weight: bold;">---</span>
<span class="Identifier" style="color: palegreen;">&gt;     which $1</span>
<span class="Identifier" style="color: palegreen;">&gt;     if [ $? -ne 0 ]</span>
<span class="Statement" style="color: khaki; font-weight: bold;">59d58</span>
<span class="Special" style="color: navajowhite;">&lt;         rm -f tmp$$</span>
<span class="Statement" style="color: khaki; font-weight: bold;">68d66</span>
<span class="Special" style="color: navajowhite;">&lt;         rm -f tmp$$</span>
<span class="Statement" style="color: khaki; font-weight: bold;">72d69</span>
<span class="Special" style="color: navajowhite;">&lt;     rm -f tmp$$</span>
<span class="Statement" style="color: khaki; font-weight: bold;">80c77</span>
<span class="Special" style="color: navajowhite;">&lt;         if grep -q "No such" tmp$$</span>
<span class="Statement" style="color: khaki; font-weight: bold;">---</span>
<span class="Identifier" style="color: palegreen;">&gt;         if [ $? -ne 0 ]</span>
<span class="Statement" style="color: khaki; font-weight: bold;">162c159</span>
<span class="Special" style="color: navajowhite;">&lt;         sudo apt-get remove 'gnuradio-*' &gt;/dev/null 2&gt;&amp;1</span>
<span class="Statement" style="color: khaki; font-weight: bold;">---</span>
<span class="Identifier" style="color: palegreen;">&gt;         sudo apt-get -y remove 'gnuradio-*' &gt;/dev/null 2&gt;&amp;1</span>
<span class="Statement" style="color: khaki; font-weight: bold;">214c211,218</span>
<span class="Special" style="color: navajowhite;">&lt;         sudo apt-get -y install $PKGLIST &gt;prereq.log 2&gt;&amp;1</span>
<span class="Statement" style="color: khaki; font-weight: bold;">---</span>
<span class="Identifier" style="color: palegreen;">&gt;         sudo apt-get -y install $PKGLIST  2&gt;&amp;1 | tee prereq.log</span>
<span class="Identifier" style="color: palegreen;">&gt;         if [ $? -ne 0 ]</span>
<span class="Identifier" style="color: palegreen;">&gt;         then</span>
<span class="Identifier" style="color: palegreen;">&gt;             echo Error occured during installing build dependency of gnuradio</span>
<span class="Identifier" style="color: palegreen;">&gt;             echo Please check your network setting and look prereq.log for details.</span>
<span class="Identifier" style="color: palegreen;">&gt;             more prereq.log</span>
<span class="Identifier" style="color: palegreen;">&gt;             exit</span>
<span class="Identifier" style="color: palegreen;">&gt;         fi</span>
<span class="Statement" style="color: khaki; font-weight: bold;">274c278</span>
<span class="Special" style="color: navajowhite;">&lt;     if [ ! -d gnuradio/gnuradio-core ]</span>
<span class="Statement" style="color: khaki; font-weight: bold;">---</span>
<span class="Identifier" style="color: palegreen;">&gt;     if [ $? -ne 0 ]</span>
<span class="Statement" style="color: khaki; font-weight: bold;">287c291</span>
<span class="Special" style="color: navajowhite;">&lt;     if [ ! -d uhd/host ]</span>
<span class="Statement" style="color: khaki; font-weight: bold;">---</span>
<span class="Identifier" style="color: palegreen;">&gt;     if [ $? -ne 0 ]</span></pre></div>