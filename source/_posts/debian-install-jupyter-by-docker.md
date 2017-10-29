---
title: 最简单的用Docker在Debian上安装Jupyter的方法
date: 2017-10-29 11:55:01
tags:
 - Debian
 - Ubuntu
 - Docker
 - Jupyter
---

# 写在前面
阅读本文之前，我假设你已经在Debian/Ubuntu上安装了最新的`docker engine`和`docker compose`，如果这倆尚未安装，请先参考这篇blog： [最简单的用Docker在Debian上安装GitLab的方法](http://wadarochi.github.io/2017/10/28/debian-install-gitlab-by-docker/) 。另外，本文描述的安装方法，在Ubuntu上应该也是可以用的。

# 安装步骤
## 获取最新的docker image
```bash
$ sudo docker pull jupyter/datascience-notebook
```

这条指令大概会下载5.5G左右的数据，下载完之后可以运行：
```bash
$ sudo docker images

...
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
jupyter/datascience-notebook   latest              312e1540f9f0        6 days ago          5.66GB
...
```

如果输出中出现`jupyter/datascience-notebook`，那就是已经下载好了

<!--more-->

## 设置外部存储
有了docker image，我们还需要设置一下外部存放Jupyter的notebook的地方：
```bash
$ sudo docker volume create --name notebook
```

## 编写供docker compose使用的配置文件
将下面的内容贴到名为`notebook.yml`的文件中去。

```yml
# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.

version: "2"

services:
  notebook:
    restart: always
    image: jupyter/datascience-notebook
    volumes:
     - "work:/home/jovyan/work"
    ports:
     - "8888:8888"
    command: >
     start-notebook.sh
     --NotebookApp.token=''

volumes:
  work:
    external:
      name: notebook
```

说明：
1. image中指定的就是我们刚刚通过`docker pull`从DockerHub上下载下来的image名字；
2. services中的volumes，以及下方的volumes，将docker中的`/home/jovyan/work`映射到了我们刚刚创建的名为`notebook`的docker volume当中，这样你重启docker的时候，里面的数据就不会消失了；
3. 请注意command中，我把Jupyter的密码给禁掉了，如果你的服务位于公网，那还是不要这么干比较好，推荐参考Jupyter官方的docker compose demo： [letsencrypt-notebook.yml](https://github.com/jupyter/docker-stacks/blob/master/examples/docker-compose/notebook/letsencrypt-notebook.yml)。

## 运行
```bash
# 我假定你已经cd到notebook.yml所在的路径了
$ sudo docker-compose -f notebook.yml up -d
```

然后你就可以打开浏览器访问你服务器的8888端口，这就是你的Jupyter。

# 想试试其它的Jupyter
Jupyter官方创建了一大堆image在 [Docker Hub](https://hub.docker.com/u/jupyter/) 上，可以随你使用、体验，比如你想用scipy-notebook，那么把上面的`jupyter/datascience-notebook`替换成`jupyter/scipy-notebook`就好了。
