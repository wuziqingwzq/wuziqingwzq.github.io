---
layout: post
title:  中间件部署常见问题
date:   2019-12-06 09:00:00 +0800
categories: middleware
tags: middleware tomcat centOS
---

汇总整理一下系统部署实施过程中经常查询的问题，避免每次再去看：

## Mysql数据库

MYSQL常见问题：
* Mysql安装位置
* Mysql初始化设置
* Mysql添加连接白名单
* Mysql设置编码
* Mysql修改端口

### Mysql安装位置

通过yum安装的mysql默认位置在``/usr/lib64/mysql``

同样可以通过``whereis mysql``加上``ls -lrt``来进行查询，具体操作可以参见JAVA安装位置的部分。

### Mysql初始化指令

这个指令可以初始化MariaDB或Mysql
```
./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql/  --datadir=/home/mysql/data
#或者使用
mysql_secure_installation
```
初始化时可以设置Root密码，是否关闭root用户远程连接权限，删除测试库。

### Mysql添加白名单

出于安全考虑，Mysql默认是不允许除了0.0.0.0之外的地址连接的，会提示连接被拒绝。可以使用以下命令允许某用户从某IP连接进来。授权root用户可以从‘192.168.1.100’IP上访问数据库，密码是‘mqozjN6hc4c3NaaI’，如果想允许所有IP访问，IP设置为‘%’

```
grant all privileges on *.* to 'root'@'192.168.1.100' identified by 'mqozjN6hc4c3NaaI' with grant option;
\\刷新用户权限表，重启数据库服务也可以
FLUSH PRIVILEGES;
```

### Mysql设置编码

Mysql数据库大部分默认安装是使用utf8编码，但是偶尔会出现编码不正确的情况，中文字符会出现乱码，最好每次安装完检查一遍相关配置。
使用``mysql -uroot -p``进入数据库后，输入以下指令：

```
\\显示数据库编码格式
SHOW VARIABLES LIKE 'character_set_%';
\\结果如下
MariaDB [(none)]>
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
```
可以看到部分编码不是utf-8，需要修改，打开mysql的配置文件

```
vi /etc/my.cnf
\\在[mysqld]下添加如下配置
collation-server = utf8_general_ci
character-set-server = utf8
```
重启服务后，再次查看编码格式，会发生变化。


### 修改Mysql服务端口
在某些情况下，服务端口被占用，需要设置其他端口，需要修改配置文件

```
vi /etc/my.cnf
\\在[mysqld]下添加如下配置
port=9999
```
重启服务后，使用``netstat -ntlp``查看端口变更情况

## JAVA

JAVA常见问题：
* JAVA安装位置
* JAVA随机数导致Tomcat过慢
JAVA环境变量

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


## tomcat

### 手动安装tomcat

偷懒的方法就是直接用yum来安装``yum install tomcat``，但是yum安装的东西版本一般都太低了。如果需要手动装的话，需要找编译好的tar包。首先在百度上搜索tomcat镜像，或者直接在[tomcat官网][tomcat]上下载，右键选择下载地址。

```
mkdir /opt/tomcat
cd /opt/tomcat
wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.50/bin/apache-tomcat-8.5.50.tar.gz
```
解压完成后，可以到tomcat/bin路径下执行startup.bat，如果启动成功，可以手动添加一个服务。

## 其他

CentOS7下的常见问题：
* 网卡配置
* 端口映射
* yum查询、安装软件
* 设置服务service文件
* chmod修改文件权限
* chown改变文件所有者

### CentOS7网络配置

一般云平台ECS租来了都是配好网络的，但是最小化的CentOS7，没有配置网卡，需要自己设置。使用``ifconfig``或者``ip addr``可以查看当前的网络状态。

```
vi /etc/sysconfig/network-script/ifcfg-en0

\\在该文件中增加指定网络配置
BOOTPROTO="static" #dhcp改为static 
ONBOOT="yes" #开机启用本配置
IPADDR=192.168.1.100 #静态IP
GATEWAY=192.168.1.1 #默认网关
NETMASK=255.255.255.0 #子网掩码
DNS1=114.114.114.114 #DNS 配置
```

### Firewalld端口映射

经常会出现这样的应用需求，将内网的端口映射到其他端口上，如果有网络安全设备的话，是很好实现的。但是在云环境中，安全设备都是由云平台底层来控制的，比如说VPC安全策略组。

实现这个需求的方法有很多，比如说使用第三方工具，安装Nginx反向代理等，功能都很强大。但是实际上有个比较容易的方法，就是用centOS自带的firewall防火墙来做端口映射。这个方法依赖于防火墙自带的IP伪装，因此开启防火墙后，需要先查询IP伪装是否开启。

```
firewall-cmd --query-masquerade #查询端口伪装状态，yes是开启
\\开启IP伪装
firewall-cmd --permanent --add-masquerade
```

IP伪装开启后，可以使用下面的指令来进行端口映射，下面的指令是将172.16.159.26的80端口映射到本机的8000端口上。
```
firewall-cmd --zone=drop --add-forward-port=port=8000:proto=tcp:toport=80:toaddr=172.16.159.26 --permanent
\\重新加载防火墙状态
firewall-cmd --reload
\\查看端口映射
firewall-cmd --list-all
```
这个方法的缺陷有两个，一是仅限TCP连接，二是防火墙服务关闭后，转发会失效。如果仅仅是本机的端口转发，则不用加IP

```
firewall-cmd --add-forward-port=port=80:proto=tcp:toport=8080 --permanent
```

### 防火墙白名单

centOS的防火墙开启后，大部分端口对外都是不开放的，需要手动添加白名单，下面是添加2222出规则的指令。

--permanent参数不加的话，服务重启后，该条配置会失效。

```
firewall-cmd --zone=public --add-port=2222/tcp --permanent
firewall-cmd --reload
```
配置完成后，使用``firewall-cmd --list-all``来查看端口映射情况

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

![linuxFile][linuxFile]

* r 代表对象是可读的（Read）
* w 代表对象是可写的（Write）
* x 代表对象是可执行的（Excute）
* - 代表无权限

chmod 赋予文件权限，可以使用 + = 来赋予和取消授权，也可以使用数字代替权限。rwx三个权限的可以转换为数字：

```
---	000	0	无权限
--x	001	1	只有执行权限
-w-	010	2	只有写入权限
-wx	011	3	写和执行权限
r--	100	4	读权限
r-x	101	5	读取和执行的权限
rw-	110	6	读取的写入的权限
rwx 111 7   所有权限
```

使用chmod可以给XXX文件修改权限：
```
chmod 777 XXX
```

### chown修改文件从属

linux系统中，对于文件权限有严格的控制，任何一个文件都属于某个用户和用户组，使用change owner（chown）指令可以更换所属的用户及用户组。

```
chown -R root:root XXXX
```

加上-R可以将下属文件的权限都修改



[Wiki-faster-tomcat]: https://cwiki.apache.org/confluence/display/TOMCAT/HowTo+FasterStartUp
[linuxFile]: /assets/pic/2019-12-06/linuxFile.jpg
[tomcat]: http://tomcat.apache.org/