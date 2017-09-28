---
title: Use Firefox and Chromium in pure ipv6 environment
tags: []
date: 2011-10-11 19:18:00
---

因为鄙校奇特的上网方式，在不用代理的情况下，每位同学同一时间只能使用一个vpn账号在一台电脑上连接外网，所以我想在某些电脑上直接使用纯ipv6网络环境，如此就可以省去部署代理带来的密钥分配的麻烦。但是非常可惜，在只有纯ipv6网络的Ubuntu下，Firefox和Chromium都是罢工的，即使在纯ipv6环境的Windows下，两者也常常罢工；同时，这也会影响纯ipv6环境中Ubuntu在线用户的设定……

通过Google搜索发现，有人还在mozilla这边提交了bug：[Cannot sign in to google account. Page stalls at waiting for account.google.com](http://support.mozilla.com/zh-TW/questions/877650)。在纯ipv6环境下的Ubuntu上面试了几次，发现只要是https协议的链接，两者都打不开，但是w3m就毫无压力，所以怀疑是Firefox和Chromium在证书方面遇到问题，于是修改两者的证书验证选项：

For firefox:
  > unselect checkbox of ``Options—&gt;Advanced—&gt;Encryption—&gt;Certificates—&gt;Validation—&gt;Use the Online Certificate Status Protocol(OCSP) to confirm the current validity of certificates''  

For Chromium:
  > unselect checkbox of ``Options—&gt;Under the Hood—&gt;HTTPS/SSL—&gt;Check for server certificate revocation''  

这样设置之后两者登陆https站点的障碍就立马消除了，Ubuntu下的在线用户也可以成功设定了，但是确实是存在安全风险的。最好的解决方案还是期待广大CA早日向ipv6过渡 ;)