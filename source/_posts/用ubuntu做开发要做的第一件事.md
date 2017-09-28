---
title: 用ubuntu做开发要做的第一件事
tags:
  - linux
date: 2010-05-03 02:23:00
---

就是将/bin/sh重新指向/bin/bash。比较干净的步骤是依次执行如下三条命令:
> <pre>sudo update-alternatives --install /bin/sh sh /bin/bash 40
> sudo update-alternatives --install /bin/sh sh /bin/dash 30
> sudo update-alternatives --config sh
> </pre>
然后按提示选择将sh重新指向bash即可

**为什么要这么做？**
为了在编译各类源码包的时候不要遇见由于操蛋的dash而引发的各式各样稀奇古怪的错误**
**