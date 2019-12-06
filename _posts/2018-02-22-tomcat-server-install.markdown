---
layout: post
title:  "Tomcat服务安装"
date:   2018-02-22 18:00:00 +0800
categories: middleware
tags: Tomcat service
---
## 设置Tomcat服务

2018年2月22日早上，项目上的一台服务器工作不正常，检查之后发现是由于系统检查补丁后自动更新重启，但是tomcat并没有运行。

![tomcat][tomcat]

以前服务器上下载的tomcat都不是安装版的，平时启动的时候，需要使用startup.bat，这样做坏处就是tomcat运行时会一直有窗口程序在桌面上，如果关闭的话，服务就停止了。并且重启启动也会导致服务停止，就像下图这样。

![startup][startup]

为了实现tomcat以服务的方式在后台运行，并且机器重启后服务也可以正常提供，需要使用tomcat自带的service.bat，这个批处理中包含了服务的安装、卸载功能。(截图中使用的是Tomcat8)

如果直接运行``C:\apache-tomcat-8.0.26\bin\service.bat``，会提示``Usage: service.bat install/remove [service_name] [/user username]``，这是需要加上服务的选项install/remove/uninstall(三选一)。

```
//C:\apache-tomcat-8.0.26是tomcat的安装路径，需要根据自己情况来修改。
//服务安装
C:\apache-tomcat-8.0.26\bin\service.bat install

//服务卸载
C:\apache-tomcat-8.0.26\bin\service.bat remove
```

如果提示``The service 'Tomcat8' has been installed.``就说明服务已经安装完成，如果提示失败，多半是由于CATALINA_HOME、CATALINA_BASE、JAVA_HOME、JRE_HOME这几个参数没有正确配置，具体的配置流程可以找百度。

在运行中输入services.msc，进入到服务配置界面，然后找到刚才安装的tomcat8服务，首先将启动类型设置为“自动”，然后将该服务启动。检查一下服务是否正常，然后再重启一下服务器，确保该服务已经开机启动了。

![services][services]

如果要卸载该服务的话，使用``service.bat remove``即可。



[tomcat]: /assets/pic/2018-02-22/tomcat.png
[startup]: /assets/pic/2018-02-22/startup.png
[services]: /assets/pic/2018-02-22/services.png