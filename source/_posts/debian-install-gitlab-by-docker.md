---
title: 最简单的用Docker在Debian上安装GitLab的方法
date: 2017-10-28 14:47:48
tags:
 - Debian
 - Ubuntu
 - Docker
 - GitLab
categories: linux
---

# 安装步骤
## 安装最新版docker engine
参考文章：[How to upgrade Docker on Debian or Ubuntu using the official source](http://ask.xmodulo.com/upgrade-docker-debian-ubuntu.html)

### 源里面的docker版本
开发服务器上使用的版本是Debian jessie，已经是老版本了，不出意料，源里面的docker果然是老版本：
```bash
$ apt search docker

lxc-docker-1.6.0/未知,now 1.6.0 amd64
  Linux container runtime
```

### 安装Docker官方源中的docker
显然太老了，那么就从docker官方源安装最新版吧，Debian指令如下：
```bash
# 添加docker官方源repo key（对Debian和Ubuntu都一样）
$ sudo apt-key adv --keyserver http://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D

# 添加docker官方源，For Debian (7.0 and higher)
$ sudo sh -c "echo 'deb https://apt.dockerproject.org/repo debian-$(lsb_release -sc) main' | cat > /etc/apt/sources.list.d/docker.list"
```

<!--more-->

如果你之前不小心装了`lxc-docker-1.6.0`，把它卸载掉：
```bash
$ sudo apt remove lxc-docker-1.6.0
```

接下来，安装最新版本的docker：
```bash
$ sudo apt-get update; sudo apt-get install docker-engine

$ docker --version
Docker version 17.05.0-ce, build 89658be
```

对于Ubuntu系统，也就是添加源的那一行指令有少许差异，把`debian-`换成`ubuntu-`就好了，具体可以参考前面列的那篇参考文章。

## 通过docker安装GitLab
参考文章：[docker-gitlab#installation](https://github.com/sameersbn/docker-gitlab#installation)

### 获取GitLab的docker image
```bash
sudo docker pull sameersbn/gitlab:10.1.0
```

如果你想尝试最新版本的GitLab，也可以把`10.1.0`改成`latest`，须谨慎。

### 让GitLab跑起来
```bash
# 用docker compose最简单，来，我们装一个docker compose
$ sudo curl -L https://github.com/docker/compose/releases/download/1.10.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 下载GitLab所需的docker-compose.yml
$ wget https://raw.githubusercontent.com/sameersbn/docker-gitlab/master/docker-compose.yml
```

在我们正式让GitLab跑起来之前，我们最好去设置一下，也就是编辑一下刚刚下载下来的docker-compose.yml，我做了如下修改：
```
47c47
<     - TZ=Asia/Kolkata
---
>     - TZ=Asia/Shanghai
53c53
<     - GITLAB_HOST=localhost
---
>     - GITLAB_HOST=xxx.xxx.xxx.xxx
```

最后，让GitLab跑起来：
```bash
$ sudo docker-compose up
```

这样你就可以通过浏览器在 http://xxx.xxx.xxx.xxx:10080 访问到你自己的GitLab了。

一切正常之后，在当前终端`Ctrl-C`终止当前的GitLab的Docker，该用下面的指令来运行GitLab Daemon就好了：
```bash
$ docker-compose up -d
```

是不是很简单，仅仅需要十条左右的无脑指令。而且之后只要你的Debian上的docker service启动，GitLab就会启动，完全无须自己去设置systemd。

## 修改设置、升级
### 你想换新版本的GitLab
```bash
# 先cd到你存放docker-compose.yml的路径，再运行下面的指令，我没看手册，不知道在其它地方运行会不会有效果
$ sudo docker-compose pull
$ sudo docker-compose up -d
```

### 你改了docker-compose.yml
```bash
# 先cd到你存放docker-compose.yml的路径，再运行下面的指令，我没看手册，不知道在其它地方运行会不会有效果
$ sudo docker-compose up -d
```

# 需求来源
讲完了安装步骤，这里扯点其它的，鄙司内把svn作为项目代码的版本管理工具，我们希望每个功能的开发，都只有一个提交，这样做有两个方面的考虑：
1. 方便做code review。
2. 如果某个功能引入bug，可以快速、准确的回退这个功能相关的代码。

有利有弊，假如只有一个提交：
1. 中间的那些修改只能本地保存了吗？万一硬盘坏了就gg了......
2. 这个功能需要多人协作怎么办？

等等，为什么不用svn分支，开发完之后merge会trunk就好了？组内很多同学也有这样的疑问。但是在我看来，直接用svn分支有如下弊端：
1. QA测试问题
2. 已有机制兼容问题

## QA测试问题
游戏公司向来特别喜欢压榨QA同学们，我也很同情他们，一个完整的开发流程如下：
```
策划提需求 --> 美术设计(optional) --> 程序开发(optional，因为有时只需要策划拖编辑器或者改表就好了) --> QA测试 --> bug修复 --> 功能放出
```

我们这还有周版本的概念，周版本就对应着每周的功能放出，这个时间点是定死的，真正的deadline，前面几个环节都可以拖，但是QA测试、bug修复一定不能拖。于是，为了避免QA同学通宵测试、程序员同学通宵改bug，QA就会在程序开发时介入测试，及早发现bug。

在开发进行中介入测试，假如此时测试的代码距离最终版有很大的区别，那基本上在最终提交之后还得把这个功能点测试一次，所以我们要尽可能避免这种区别，那么导致这种区别的原因是什么呢？
1. QA测试到的代码不是基于trunk HEAD的（这种情况比较多），那么如果你用了svn分支，当你需要同步到trunk HEAD，老老实实去sync merge吧，但是如果你没有用svn分支，`svn update`就好了（两者都可能需要解决冲突，但是后者明显会简单得多，对新手也不容易错）。
2. 程序员自己在测试完之后又去重构了（这种情况比较少，一般是因为code review）。

所以我觉得从“让QA测试到最终代码方面”考虑，不用svn分支会更加好一些。

## 已有机制的兼容问题
### code review问题
用了svn分支，你的diff是通过比对两个svn分支来生成的；没有用svn分支，你的diff只需要`svn diff`就好了。后者的话，组内正在使用的review board工具就不需要修改。（最弱的理由......）

### 代码同步机制
一直以来，我们在内服开发时，所有客户端会首先把代码、资源会同步到trunk HEAD，然后再同步特定功能的代码和资源——我们称为“功能分支”。功能分支本身的生成则依赖`svn diff`和git：首先用`svn diff`找到所有相对trunk HEAD的修改，然后把这些文件扔到git中去，会有一个工具根据git去生成功能分支的内容。

从“已有机制的兼容”考虑，也是不用svn分支会更加好一些。

然后我们既然已经引入了git，何不干脆就用git来做svn的补充就好了，为了防止硬盘挂掉导致本地git repo也消失的风险，同时也是多人协作的需要，我们需要一台git服务器，GitLab不错，于是有了前面的教程。

## svn、git混用的建议
* 更新的时候先更新git再更新svn
* 保险起见，每个功能开发都新建一个新的git repo
