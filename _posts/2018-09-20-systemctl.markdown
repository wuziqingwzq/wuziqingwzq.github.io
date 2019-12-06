---
layout: post
title:  "systemctl指令和service指令"
date:   2018-09-20 09:20:00 +0800
categories: middleware
tags: CentOS Linux
---
## 前言
之前接触Linux系统较少，都是面向搜索引擎进行操作，很多指令都是复制过来直接粘贴，其中一个很常用的就是启动服务。

```
service network restart
```
重启网络服务，但是有时候发现网上教程的指令也会使用这个指令``systemctl restart network``，于是问题来了，这两个指令都是重启某个服务，区别在哪儿。


百度之后发现，Linux系统在开机时完成初始化的最后一步，会启动系统的第一个进程（PID为1），在这个进程，在一些老版本的Linux中，会启动一个叫做SysVinit的进程，用于管理其他进程。而在一些新的Linux系统中，则是启动systemD（还有一些是UpStart），缩写可以看作是System Daemon（系统守护进程）。

在使用SysVinit的系统中（比如CentOS6.X），只能使用service工具来控制系统服务，在使用SystemD的系统中（比如CentOS7.X），则使用一个新的工具——systemctl来控制进程与服务，它整合了service和chkconfig的功能。当然，你也可以在CentOS7中使用service指令，两者之间并不冲突。

## service工具
service的用法是可以通过``service -h``来查看，会弹出用法：
```
Usage: service < option > | --status-all | [ service_name [ command | --full-restart ] ]
//差不多等于:
service(选项)(参数)
//比如service network restart
//参数可以选择start stop restart status等
```
在执行service指令时，会调用/sbin/service这个shell脚本，并将参数传给/etc/init.d/下的相关文件，因此在执行service提示找不到相应服务的时候，应该检查一下/etc/init.d/下面是否有相应的启动文件。

另外需要说一点，chkconfig指令也是基于SysV系统的，和service一样。

## systemctl工具
以下内容是基于linux中国论坛的帖子，更详细的指令可以参考这篇[systemctl 命令完全指南][linux中国]

之前有说过，systemctl是基于systemD工具的，所以要确定自己系统中是否可以使用systemctl指令，可以先查询systemD的版本：

```
systemctl --version
//一般会提示systemd XXX，XXX就是你的systemD版本。
```

systemctl会在以下两个路径下寻找service的脚本，如果systemctl指令提示你不存在这个unit，那么可能是下面这两个路径还不存在你需要运行服务的脚本。
```
/etc/systemd/system
/usr/lib/systemd/system
```

对于systemctl工具，我们常用的指令主要有：
```
systemctl start network    //启动服务
systemctl stop  network    //停止服务
systemctl restart network  //重启服务
systemctl enable network   //自启动服务
systemctl disable network  //禁止自启动
```
## 结论
通过网上的资料，最后得出的结论是，在支持systemD的系统中，service和chkconfig已经过时了,使用他们获得的服务列表是不完全的，而systemctl也兼容了它们的功能，最好只使用systemctl工具。



[linux中国]: https://linux.cn/article-5926-1.html