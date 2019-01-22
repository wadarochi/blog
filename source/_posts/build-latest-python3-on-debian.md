---
title: Debian 8.9(Jessie)编译Python 3.7.2
tags:
	- debian
	- vim
	- python3
	- gcc
---
# 前言
本文涉及的所有vim插件，在这里均有列出：[pt.vim](https://github.com/wadarochi/pt.vim)。

# 问题起源
猪厂内网，本人某一台开发服务器上目前还运行着古董Debian：Jessie，官方repo中各种legacy软件让人崩溃。所以我在home路径下放了个.local，把各种常用软件都编译了一份最新版扔在这里，比如：
* vim 8.1.789——为了体验8.1引入的terminal功能
* tmux 2.8——因为vim material主题在老版本的tmux中会颜色错乱
* rg——Debian Jessie repo中没有这货
* mc——Debian Jessie repo中这货版本太老了
* python 3.7.2——好用的vim插件denite.nvim要求Python 3.6以上，而官方repo中只有3.4
* gcc 8.2——编译Python 3.7需要新版本的gcc，索性build一个当下最新版本


某天早上，我在开发服务器上的vim中例（手）行（贱）运行更新了所有vim插件：
```vim
:call dein#update()
```

然后，denite.nvim就不能用了：当我例行`Ctrl-P`，期望打开当前路径下某一文件时爆了一个Python trace。一个Python 3.6才引入的新方法报错了——找不到。打开denite.nvim的git log一看，果然作者有了新想法（我也喜欢追新，比如`--std=c++latest`）：

```txt
* [2018-10-21] [9ba61c7] | Python 3.5+ is required {{Shougo Matsushita}}
```

因为这个插件很好用，所以只能升级开发机上的Python了。上文也说了，repo中只有3.4，搜索了各种backport，都没有为Jessie提供3.5以上的Python3，遂动手从源码编译。

# 编译Python 3.7.2
## 准备工作——编译gcc 8.2
Debian Jessise repo中的gcc 4.9.2无法胜任编译Python 3.7.2的重任，所以需要先从源码中build一份gcc，反正都已经从源码build了，干脆来个最新的，本文写作时，GNU FTP上最新的gcc为8.2.0，就是它了。

```zsh
# prepare dev libs
sudo apt install libgcc-4.9-dev

# prepare build directory
mkdir gcc
cd gcc
mkdir gcc-build

# download
wget https://ftp.gnu.org/gnu/gcc/gcc-8.2.0/gcc-8.2.0.tar.xz

# extract gcc source code
tar xz gcc-8.2.0.tar.xz
cd gcc-8.2.0

# prepare dependencies
./contrib/download_prerequisite

# configure
../gcc-8.2.0/configure --prefix=/home/xxx/.local --enable-shared --disable-multilib --enable-threads=posix --enable-__cxa_atexit --enable-clocale=gnu --enable-languages=c,c++,fortran,go,objc,obj-c++

# build
make -j4

# install into my home directory
make install
```

因为服务器是64位系统，所以并不需要编译32位版本的gcc，于是在build gcc的时候将multilib这个特性给禁用了。编译gcc花了我接近两个小时......在继续往下之前，请确保`/home/xxx/.local/bin`已经在你的`PATH`当中。

```zsh
>> g++ --version
g++ (GCC) 8.2.0
Copyright (C) 2018 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

## 编译Python
```zsh
# prepare build directory
mkdir python
cd python
mkdir python-build

# download
wget https://www.python.org/ftp/python/3.7.2/Python-3.7.2.tar.xz

# extract
tar -xf Python-3.7.2.tar.xz

# configure
../Python-3.7.2/configure --enable-optimizations --prefix=/home/xxx/.local

# build
make -j4

# install
make install
```

测试一下：
```zsh
python3 -c "import sys; print(sys.version)"
3.7.2 (default, Jan 22 2019, 17:11:23)
[GCC 8.2.0]
```

Cool!

## 重新编译vim
上面的步骤全部搞定之后，正常情况下，在vim里面运行Python插件的时候应该已经可以找到`PATH`中的Python 3.7.2了，但是如果和我一样，之前编译vim的时候指定了系统的Python 3.4，那么你就得重新编译vim。

```zsh
# clone source code of vim from github
git clone https://github.com/vim/vim.git

# configure
export LDFLAGS="-rdynamic"
PYTHON="/home/xxx/.local/bin/python3.7" ./configure --with-features=huge --enable-python3interp=yes --enable-fail-if-missing --enable-multibyte --enable-gui=no --enable-cscope --prefix=/home/xxx/.local/

# build
make -j4

# install
make install
```

请注意这里一定要`export LDFLAGS="-rdynamic"`，原因见这里：[undefined symbol: PyByteArray_Type #3629](https://github.com/vim/vim/issues/3629#issuecomment-440845680)。

大功告成之后，denite.nvim工作又一切正常了，开心！只是如果你一直在vim源码路径编译的话，修改configure之前记得清理一下configure的中间文件：
```zsh
git clean -fd
git clean -fX
```

btw：写这篇文章的时候正好来了两位外国主播友人，给他们介绍了一番Rules of Survival新东东，好像我英语也还行，哈哈哈。
