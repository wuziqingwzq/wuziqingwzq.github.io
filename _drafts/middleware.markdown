---
layout: post
title:  中间件部署常见问题
date:   2019-12-06 09:00:00 +0800
categories: middleware
tags: middleware tomcat centOS
---


记录一下实施过程中经常在查询的问题：

MongoDB添加白名单IP
Redis添加白名单IP
Service配置无法执行Java
Linux 目录结构
Linux 授权chmod 和chrown
CentOS


## Mysql数据库

MYSQL常见问题：
* Mysql安装位置
* Mysql初始化指令
* Mysql添加白名单IP
* Mysql设置编码
* yum安装指定版本软件
* yum安装且下载指令

### Mysql安装位置

通过yum安装的mysql默认位置在``/usr/lib64/mysql``

### Mysql初始化指令

这个指令可以初始化MariaDB或Mysql
```
./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql/  --datadir=/home/mysql/data
#或者使用
mysql_secure_installation
```
初始化时可以设置Root密码，是否允许远程连接，删除测试库等

## JAVA

JAVA常见问题：
* JAVA安装位置
* JAVA随机数导致Tomcat过慢
* JAVA环境变量

### JAVA安装位置

使用yum安装JAVA后，会默认配置好环境变量，但是有时候还需要手动配置这个参数。一般来说，JAVA安装在``/usr/java/jdk-XXXX/``下，根据JAVA版本后缀有所不同。

查询JAVA安装位置，使用``whereis java``可以查看java可执行文件的路径，然后在查询这个软连接的实际位置。

```
whereis java
输出：java: /usr/bin/java
ls -lrt  /usr/bin/java
输出：/usr/bin/java -> /etc/alternatives/java
ls -lrt /etc/alternatives/java
输出：/etc/alternatives/java -> /usr/java/jdk-9.0.1/bin/java
```

重点就是查看软链接指向的源文件。

### JAVA随机源修改

在Tomcat中，有时候会出现启动非常慢（无应用，启动5-10分钟以上）的情况，是因为JAVA随机源的问题。在Tomcat&+启动时会依赖JAVA产生随机数，如果JAVA使用的随机源是默认的``/dev/random``设备，该设备是阻塞式的，如果外界的环境噪声不足，产生噪声过慢，就会导致获取随机数速度非常慢。

我们可以采取降低安全性的方式，采用非阻塞的随机数发生器``/dev/urandom``来解决这个问题。修改JAVA配置文件：

```
vi /%JAVA_HOME%/jre/lib/security/java.security 
```

找到其中的securerandom.source这一项进行修改：

```
securerandom.source=file:/dev/urandom
```

这是一项变通的做法，但是并不安全。按照网上的资料，JDK8+优化了这个问题。其他能够加速Tomcat启动速度的方法可以参考Wiki：[Wiki-faster-tomcat][Wiki-faster-tomcat]

## CentOS7

网卡配置
yum查询、安装软件
服务service文件的常用设置
chmod修改文件权限
chown改变文件所有者


### CentOS7网络

一般云平台ECS租来了都是配好网络的，但是最小化的CentOS7，没有配置网卡，需要自己设置。使用``ifconfig``或者``ip addr``可以查看当前的网络状态。

```
vi /etc/sysconfig/network-script/ifcfg-en0
```

### YUM安装指定版本的包

使用以下指令可以查询安装的RPM包

```
#当前安装RPM
rpm -qa |grep XXX
#可以安装的包
yum search XXX
yum list|grep XXX
```

### 设置服务自启动

有的时候需要部署的包是SpringBoot开发的jar，而不是War包，正常的启动指令是：

```
nohup java -jar XXX.jar &
```
这样就在后台运行了，但是如果出现机房断电或者服务器异常重启等问题，还需要手动启动服务，因此需要把他加到systemctl中，并且自启动。

service文件需要自己写，在以下两个路径下都可以添加：

```
#用户路径
/usr/lib/systemd/system
#系统路径
/etc/systemd/system
```

test.service文件内容如下：
```
[Unit]
Description=Test JAVA service

[Service]
WorkingDirectory=/opt/Test
User=root
Type=forking
ExecStart=/opt/signature/Startup.sh
ExecStop=/opt/signature/Shutdown.sh
Restart=on-failure
PrivateTmp=true
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Description是描述、WorkingDirectory是工作路径，ExecStart中的sh需要自己写启动脚本。

### 修改文件权限

在Linux系统中，文件系统包含以下内容：用户、用户组、文件权限。一个文件会属于一个用户及用户组。使用ll查看后，可以看到有10个字符，第一个表示文件类型，后面9个字符三个为一组，分别代表所属用户、所属组用户、非所属组用户的权限。

* r 代表对象是可读的（Read）
* w 代表对象是可写的（Write）
* x 代表对象是可执行的（Excute）
* - 代表无权限

chmod 赋予文件权限，可以使用 + = 来赋予和取消授权，也可以使用数字代替权限。rwx三个权限的可以转换为数字：

---	000	0	无权限
--x	001	1	只有执行权限
-w-	010	2	只有写入权限
-wx	011	3	写和执行权限
r--	100	4	读权限
r-x	101	5	读取和执行的权限
rw-	110	6	读取的写入的权限
rwx 111 7   所有权限

使用chmod可以给XXX文件修改权限：
```
chmod 777 XXX
```


## TOMCAT











[Wiki-faster-tomcat]: https://cwiki.apache.org/confluence/display/TOMCAT/HowTo+FasterStartUp
[linuxFile]: daibuchong 