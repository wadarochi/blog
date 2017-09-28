---
title: Boost threads unavailable on this platform
tags:
  - C_C++
  - linux
  - ubuntu
date: 2012-10-15 21:47:00
---

在Ubuntu 11.04平台编译GNU Radio第三方的gr-chancoding模块时，出现了这个莫名的错误。主要的错误信息有：
> Error: #error "Threading support unavaliable: it has been explicitly disabled with BOOST_DISABLE_THREADS"
> Error: #error "Sorry, no boost threads are available for this platform."
> Error: #error "Boost threads unavailable on this platform"当时我就错愕了，Ubuntu在不济，也不至于改到失去Boost的线程能力吧...肯定是我打开的方法错了。Google搜索标题所示的错误信息之后，看到了这个链接：[使用 GCC 4.7 编译 Passenger](http://blog.pinepara.info/tech/compile-passenger-with-gcc-4-dot-7/)得到如下解释：
> 根据其提供的信息显示该BUG出现的原因是GCC 4.6之前都定义了GLIBCXX_HAVE_GTHR_DEFAULT，而 GCC 4.7 之后改为定义_GLIBCXX_HAS_GTHREADS。这个改动破坏了Boost代码中的判断逻辑，从而错误的定义了 BOOST_DISABLE_THREADS，而非 BOOST_HAS_THREADS。此刻才恍然大悟，之前为了体验C++11各种特性，在这台Ubuntu 11.04上通过launchpad源安装了GCC 4.7，导致了Boost傲娇...
于是，用update-alternatives把g++改回4.5，一切就正常了。