备忘录：


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

### Mysql初始化指令

这个指令可以初始化MariaDB或Mysql
```
./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql/  --datadir=/home/mysql/data
```

## JAVA

JAVA常见问题：
* JAVA安装位置
* JAVA随机数导致Tomcat过慢
* JAVA环境变量

### JAVA安装位置（待验证）

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

### YUM安装









[Wiki-faster-tomcat]: https://cwiki.apache.org/confluence/display/TOMCAT/HowTo+FasterStartUp