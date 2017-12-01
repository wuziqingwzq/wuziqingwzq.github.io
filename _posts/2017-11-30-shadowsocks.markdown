---
layout: post
title:  "ShadowSocks影梭部署教程"
date:   2017-11-30 21:00:00 +0800
categories: SS/SSR
tags: ShadowSocks SS Linux CentOS
---

## 影梭ShadowSocks是什么
影梭ShadowSocks（下文简称SS）是一款基于Socks5代理方式的网络数据加密传输软件。主要用处是通过特定的中转服务器完成数据传输，可以用于科学上网。

## SS与VPN的区别是什么
简单来说，VPN是一种虚拟网络，搭建之后所有的流量都会从该虚拟网络走，服务端到客户端搭建起来一条长连接的隧道。而SS是基于socks5协议的，只是单纯的转发数据包，这样的好处是访问国内网站时速度较快，而访问受拦截的网站时则可以通过中转服务器发起请求。

## 优点与缺点
由于我也是刚接触这款工具，因此理解可能不够深刻。但是通过查询相关资料来看，SS的优缺点如下：

* 优点：

可以通过SS客户端配置PAC模式，制定在访问某些网站的时候才使用转发，可以减少流量的使用。并且不会影响到国内网站的访问速度。

中间人可以通过通讯的协议特征来进行拦截封锁，而SS可以选择混淆模式，降低被拦截的几率。

* 缺点：

由于SS只使用预定密钥作为客户端和服务器的唯一辨别身份方式，很容易受到攻击，甚至暴力破解密码，需要额外的安全配置。

***

## 服务器的购买
SS分为客户端和服务端，如果用于科学上网，则服务端需要拥有一个能被访问的国外IP。如果你还没有境外的服务器的话，需要先租用一台，从网上搜到的情况每月价格在2.5刀至5刀不等，折合人民币17-33元左右。

以下有几个推荐的服务提供商：
1. [Vultr](https://www.vultr.com/?ref=7262500)，每月最低2.5$，但是需要选地区，有些地区最低5$。
2. [搬瓦工](https://bwh1.net/)，最低19.99一年（平均每月1.67$）
3. [DigitalOcean](https://www.digitalocean.com)
其实网上还有很多其他的，可以自己搜索一下，但是机房一定要在国外。

我使用的Vultr的服务器，现在看起来还比较稳定。进入网站之后，需要注册一个账户，登陆之后界面如下。
![vultr][vultr]
左边的菜单分别是
* Servers服务器：用于管理当前的云服务器或者新增示例。
* Billing账单：充值页面，可以使用支付宝，好评。可以查看充值记录以及余额等
* Support技术支持：常见问题、文档以及开发票等。
* Affiliate推广：你可以分享链接让别人注册，可以获得优惠卷。比如上面我推荐的链接中就带有我的邀请码。
* Account账户：本账户的资料修改等。

### 选择配置

点击Servers，可以新增一台服务器
![servers][servers]
1. 选择服务器所在地，有东京、新加坡、伦敦、法兰克福、巴黎、芝加哥等，部分地区的机房是没有2.5$/month的服务器的，需要注意一下
2. 选择服务器类型，我选择的是CentOS7，也可以选Debian、windows等
3. 选择服务器的配置，我选择的是最便宜的2.5$一个月的，性价比最高，因为SS是轻量级的应用，对于配置要求很低。500G每个月的流量也足够使用了。
4. 最下面选择Deploy Now就可以生成服务器示例了。如果你没有充值的话，这个地方会提醒充值。

### 连接服务器
还是选择Servers，里面会出现一个列表，并且还有一个互联网IP，比起国内的云服务器提供商把IP和服务拆开卖良心太多了。

服务器初始化之后，可以使用SSH客户端远程连接，我使用的是Putty。


## 服务端配置

### 安装配置
在CentOS-7下，推荐使用PIP快速安装服务，免去了配置与编译的工作。PIP是python版本的包管理工具，因此环境依赖中需要先安装python。

使用以下指令安装PIP（第一句是安装自动补全代码的插件，非必要）

```
yum install bash-completion
curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
python get-pip.py
```

可以使用以下指令测试PIP安装是否正常，如果弹出版本相关的信息则安装成功
```
pip -V
# 正常会显示：pip 9.0.1 from /usr/lib/python2.7/site-packages (python 2.7)
```

下一步是安装shadowsocks的服务端
```
pip install --upgrade pip    
pip install shadowsocks
```

安装完成之后，需要对服务器进行配置。通常会在/etc/shadowsocks.json该位置添加配置文件。
```
nano /etc/shadowsocks.json
```
添加文件内容为：
```
{
    "server":"0.0.0.0",
    "server_port":8989,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"yourpassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```
其中需要修改的内容有server_port服务器端口与password密码。按ctrl+X保存退出。

### 启动服务

现在测试一下启动,如果提示started则启动完成。
```
/usr/bin/ssserver -c /etc/shadowsocks.json -d start
```

同样的，停止服务与重启服务的指令如下：
```
# 停止服务
/usr/bin/ssserver -c /etc/shadowsocks.json -d stop
# 重启服务
/usr/bin/ssserver -c /etc/shadowsocks.json -d restart
```
服务启动好之后，我们就可以直接在手机或者电脑上下载相应的客户端软件进行连接了。建议在Github上直接下载


## 客户端配置
### Windows
在百度搜索Shadowsocks好像已经找不到下载链接了，但是仔细找找应该还是有的。建议在[***github***][github-windows]上直接下载。进入该页面后，有一个下载地址。
![githubpage][githubpage]

下载的压缩包解压运行后界面如下
![ss][ss]

需要填写的内容有服务器地址、服务器端口、连接密码与加密方式，这一部分内容在服务器上的配置文件里面有。右键右下角的飞机图标可以选择服务器以及代理方式等，通常选用PAC模式，不会影响国内网站的速度。

启动之后，可以在浏览器中尝试一下，应该就可以访问你想访问的网站了

### IOS
在APPSTORE中搜索SSR,其中的SsrConnect、shadowrocket等软件都可以用于连接SS服务器。配置和Windows类似。

# 防火墙配置
服务启动了，配置好客户端之后，无法上网？

可能是由于防火墙将SS服务的端口给拦截了，最简单的检查方法就是关掉防火墙，看看服务是否恢复。

* 检查服务器是否开启了防火墙
需要注意的是，在本小节中提到的关闭防火墙的指令是针对于CentOS7的，CentOS6.5以下的版本可能不支持，原因是CentOS7开始使用了firewall作为防火墙，而低版本是使用的iptables。

首先检查防火墙状态
```
firewall-cmd --state
# 未运行——not running
# 运行————running
```

如果防火墙正在运行，那么可以关闭服务器防火墙试试
```
# 关闭防火墙
systemctl stop firewalld.service
# 启动防火墙
systemctl start firewalld.service
```

然后在客户端上使用全局代理模式访问网页，如果可以打开，说明确实是防火墙影响了该服务。较粗暴的方式是直接关闭防火墙，当然也可以配置防火墙开发你的SS服务端口。

停止防火墙服务
```
# 禁用防火墙服务
systemctl disable firewalld.service
# 启用防火墙服务
systemctl enable firewalld.service
# 检查服务状态
systemctl list-unit-files|grep firewalld.service
```

***

## 参考文档
[图文教程：手把手教你使用 VPS 服务器搭建 Shadowsocks](https://www.rkdot.com/shadowsocks-server-setup/)



[github-windows]: https://github.com/shadowsocks/shadowsocks-windows/releases
[githubpage]: /assets/pic/2017-11-30/githubpage.png
[ss]: /assets/pic/2017-11-30/ss.png
[vultr]: /assets/pic/2017-11-30/vultr.png
[servers]: /assets/pic/2017-11-30/servers.png

