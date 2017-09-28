---
title: 推荐一个简单方便的条形码识别库--ZBar
tags:
  - barcode
date: 2010-05-10 01:46:00
---

最近莫名奇妙的突然突入到各式各样的领域中，比如生成条形码，识别条形码，编造、解析GPS帧格式，经纬度与直角坐标变换，小波变换求特征值，心电图数据查询...

然后觉得应该把这些东西记下来

What is ZBar?
ZBar is a bar code scanning and decoding library.

ZBar支持识别二维条形码，纯C实现(可选C++封装)，提供了C/C++、python、perl语言的接口，可以用在iPhone上...

在官方wiki页面(http://sourceforge.net/apps/mediawiki/zbar/index.php?title=HOWTO:_Scan_images_using_the_API)有详细的用法

话说这个库什么都好，就是wiki里面提供的例子全是用的Magick++，用Windows平台+VC6的童鞋可能要郁闷了。

其实不用Magick++也是可以的，ZBar的文档说传给parser处理的图片的格式最好是Y800(http://fourcc.org/yuv.php#Y800)的，使用OpenCV的函数cvCvtColor(cimg,gimg,CV_BGR2GRAY)也是可以搞定的，不相信的童鞋可以去看看这个函数的文档，对比一下CV_BGR2GRAY生成的图像是不是就是Y800格式

如果是在Linux下面，还是用Magick++比较好