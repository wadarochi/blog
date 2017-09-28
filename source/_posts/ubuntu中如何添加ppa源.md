---
title: ubuntu中如何添加ppa源
tags:
  - ppa
  - ubuntu
date: 2011-04-19 21:36:00
---

1.  找到你需要的软件在``launchpad.net''上的页面，比如用``Google''搜索``ubunut Firefox ppa''，能找到如下页面：[firefix ppa](https://launchpad.net/%7Eubuntu-mozilla-daily/+archive/ppa)2.  在页面上寻找下图红框中所示的``**ppa:ubuntu-mozilla-daily/ppa**''：

    [![](/img/demo-small.png)](/img/demo.png)

3.  在``ubuntu''中打开终端，并且输入如下命令：> sudo apt-add-repository ppa:ubuntu-mozilla-daily/ppa> sudo apt-get update> sudo apt-get upgrade
