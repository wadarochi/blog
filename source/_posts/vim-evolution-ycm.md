---
title: vim plugin evolution -- ycm
tags:
  - vim
  - YouCompleteMe
date: 2018-10-14 15:12:00
---

# 前言
因为工作关系，本猪厂程序员常年在多个操作系统[^1]和语言[^2]，框架[^3]之前频繁切换，所以尽管已经有强如PyCharm和Android Studio这样好用的免费IDE可供使用，还是离不开Vim．

[^1]: 目前每天工作中用到的操作系统： Windows 10、MacOS、Debian、FreeBSD(谢天谢地，最近已经用不到了，奈何猪厂已上线的游戏就算没几个人玩也要持续维护很久，还得留着开发机)、Ubuntu(自用，btw: 猪厂自研游戏引擎支持Linux)

[^2]: 时不时在下述语言之间切换：Python, lua, c/c++, objective-c, c#, java, groovy, javascript(自用)

[^3]: 框架更是数不胜数，从自研游戏引擎，开源游戏引擎到各种服务端框架应有尽有，所以想体验彻底的全栈的感觉的哥们姐们，可以来猪厂游戏部门试试:)

回归正题，既然Vim不得不用，被PyCharm和Android Studio宠坏了之后，就总会不自觉想把Vim变得和它们一样好用．或者更好一些;)

这个系列的文章主要记录采用各种Vim插件在两个方面增强Vim的使用体验：
* 自动补全
* 搜索、导航
 - 文件
 - 符号
 - 调用/被调用(Caller and Callee)
 - 文本

最后也会记录一下其它小功能，比如显示缩进字符，pattern match次数什么的．

<!--more-->

# 自动补全
## YouCompleteMe
在YouCompleteMe出现之前，我试过非常多的Vim自动补全插件：
* 基于ctags的[OmniCppComplete](https://www.vim.org/scripts/script.php?script_id=1520)
* 基于gcc的[gccsense](https://github.com/m2ym/gccsense)
* 基于clang的[clang_complete](https://github.com/Rip-Rip/clang_complete)

没有对比就没有伤害，如果没有感受过Visual Assist那无比流畅和智能的补全效果，估计我会一直使用OmniCppComplete．可是偏偏被Visual Assist震撼过了，跑到BBS上讨论了一番，大家觉得，如果没有编译器支持的话，Vim想要获得接近Visual Assist的效果都很难．

Visual Assist官方演示视频：
<iframe src="//www.youtube.com/embed/0hOztCPEi-0" frameborder="0" allowfullscreen=""></iframe>

后来，我先后发现了gccsense和clang_complete，先说gccsense，安装比较麻烦，使用条件比较苛刻：
* Objective-C and Java or else which is supported in GCC is not supported
* Development assist is not supported in a code which can not be compiled with GCC 4.4
* Development assist is not available in header files.

后者可用，可惜仅支持c/c++这两门（一门？）语言，并且渐渐的也步了gccsense的后尘，不怎么维护了．

再后来就遇到[YouCompleteMe](https://github.com/Valloric/YouCompleteMe)，迄今为止(2018年10月)还有上百位开发者在积极的开发功能．

以下是官方介绍：
> YouCompleteMe is a fast, as-you-type, fuzzy-search code completion engine for Vim. It has several completion engines:
> * an identifier-based engine that works with every programming language,
> * a Clang-based engine that provides native semantic code completion for C/C++/Objective-C/Objective-C++/CUDA (from now on referred to as "the C-family languages"),
> * a Jedi-based completion engine for Python 2 and 3,
> * an OmniSharp-based completion engine for C#,
> * a combination of Gocode and Godef semantic engines for Go,
> * a TSServer-based completion engine for JavaScript and TypeScript,
> * a racer-based completion engine for Rust,
> * a jdt.ls-based experimental completion engine for Java.
> * and an omnifunc-based completer that uses data from Vim's omnicomplete system to provide semantic completions for many other languages (Ruby, PHP etc.).

具体的安装和使用方法我就不写啦，官方的文档写的非常详细，照着做就好了，Windows平台不想自己编译可以去下载[韦一笑编译版本](https://www.zhihu.com/question/25437050/answer/95662340)．这里我主要总结一下最蛋疼的C-Family自动补全配置．

## compilation database for C-Family auto completion
使用YCM[^4]开发C-Family代码的时候有一个非常麻烦的地方－－需要`.ycm_extra_conf.py`，因为YCM借助了clang的力量，而这个过程中clang要知道怎么编译你当前编辑的文件(`compilation unit`)．

[^4]: YCM是YouCompleteMe的缩写

`.ycm_extra_conf.py`内容多而且杂，不建议手写．要是每个工程都手写一遍能疯．github上有一个Vim插件项目可以帮助生成`.ycm_extra_conf.py`文件－－[YCM-Generator](https://github.com/rdnetto/YCM-Generator)，但是很可惜，不是很好用，因为它在选cmake作为build system的时候总是漏掉各种参数，而选用make作为build system，要重新build一遍而且大概率生成失败，另外作者目前也不准备继续开发了．

幸运的是，这个需求不仅仅来源于YCM，比如Lint类的代码检查工具也需要借助编译器的力量，也需要提供`compilation database`，目前已知的生成`compilation database`的方法有：
* cmake
* Bear
* xcpretty

### cmake
cmake如果将generator设置为`Unix Makefiles`或者`Ninja`，在生成工程的时候将`CMAKE_EXPORT_COMPILE_COMMANDS`设置为`ON`，就可以生成名为`compile_commands.json`的`compilation database`．

依据Prof. Dr. Karsten Borgwardt的blog: [YouCompleteMe and CMake](http://bastian.rieck.ru/blog/posts/2015/ycm_cmake/)，对最外层的CMakeLists.txt做如下修改，即可以生成`compile_commands.json`，并且每次重新cmake之后可以按需更新源码路径下的`compile_commands.json`．
```diff
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -4,6 +4,8 @@ project (Tutorials)

 find_package(OpenGL REQUIRED)

+set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
+

 if( CMAKE_BINARY_DIR STREQUAL CMAKE_SOURCE_DIR )
	 message( FATAL_ERROR "Please select another Build Directory ! (and give it a clever name, like bin_Visual2012_64bits/)" )
@@ -773,3 +775,9 @@ elseif (${CMAKE_GENERATOR} MATCHES "Xcode" )

 endif (NOT ${CMAKE_GENERATOR} MATCHES "Xcode" )

+if( EXISTS "${CMAKE_CURRENT_BINARY_DIR}/compile_commands.json" )
+  execute_process( COMMAND ${CMAKE_COMMAND} -E copy_if_different
+    ${CMAKE_CURRENT_BINARY_DIR}/compile_commands.json
+    ${CMAKE_CURRENT_SOURCE_DIR}/compile_commands.json
+  )
+endif()
```

对比较新版本的YCM，此种情况下不再需要`.ycm_extra_conf.py`：
> YCM looks for a file named 'compile_commands.json' in the directory of the
> opened file or in any directory above it in the hierarchy (recursively); when
> the file is found, it is loaded.

### Bear
[Bear](https://github.com/rizsotto/Bear)，虽然只支持GNU make，但是用起来真是太简单了，比如：
```bash
bear make -j32
```

编译成功之后，当前路径下会生成`compile_commands.json`．

有趣的是，Bear的工作原理是直接hook住build命令的exec方法解析参数：
> Bear executes the original build command and intercepts the subsequent execution calls.  To achieve that Bear uses library preload mechanism provided by the dynamic linker.  There is a library which defines the  exec  methods
> and used in every child processes of the build command.  The executable itself sets the environment up to child processes and writes the output file.

btw: Bear源码中并没有限定死GNU make，理论上对cmake之外的build system应该都有效，说不定对xcodebuild也有效，只是需要注意：[Bear on OSX](https://github.com/rizsotto/Bear/wiki/Usage#osx)．当然，接下来会说明我们在MacOS上的其它选择．

### xcpretty
[xcpretty](https://github.com/supermarin/xcpretty)，是一个处理xcodebuild输出的工具，处于持续开发中，号称比xcodebuild更新还要快，还要超前．

安装：
```zsh
gem install xcpretty
```

使用方法：
1. 首先clean你的XCode工程；
2. `xcodebuild -project PROJECT_NAME.xcodeproj | xcpretty -r json-compilation-database --output compile_commands.json`；
3. 把生成的`compile_commands.json`扔到工程源码的根目录．

## swift
YCM并不支持swift，于是[Jerry Marino](https://github.com/jerrymarino)自己写了[iCompleteMe](https://github.com/jerrymarino/iCompleteMe)：
> iCompleteMe is based on YouCompleteMe. After spending a over a year attempting to implement Swift support for YouCompleteMe, I found that it wasn't possible to achieve ideal behavior under the conventions of YCM; iCM spawned.

iCompleteMe可以与YCM共存，非常喜闻乐见的是，他还顺手写了一份解析XCode log生成可供iCompleteMe使用的`compile_commands.json`的工具：[XcodeCompilationDatabase](https://github.com/jerrymarino/XcodeCompilationDatabase)．

# 总结
本文主要列举了：
1. 选择YCM的心路历程；
2. YCM配置C-Family自动补全可以使用的一些工具，它们能极大的减轻配置的工作量；
3. 用于swift，基于YCM的iCompleteMe．

如果哪位发现本文由谬误或者由更好的推荐，欢迎留言或者邮件联络;)
