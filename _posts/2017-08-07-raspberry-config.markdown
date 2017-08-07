---
layout: post
title:  "树莓派学习（三）：常用设置"
date:   2017-08-07 11:53:00 +0800
categories: RaspberryPi
tags: raspberry raspbian
---
使用VNC连接树莓派时，有一些常用设置，本文中列举了以下设置：
* 修改账户密码
* 修改桌面壁纸
* 切换中文环境以及编码方式
* 增加中文输入法
* 设置固定IP
* 新增WiFi连接

## 修改账户密码
在每次连接VNC以及SSH的时候，如果一直使用pi与raspberry的默认组合，系统会提示不安全，建议修改密码。修改密码在左上角点击开始菜单（左上角）——Preferences（首选项）——Raspberry Configuration（树莓派配置）——System(系统)——Change PassWord（修改密码）

![修改密码][修改密码]

## 修改桌面壁纸
在2017年7月这个版本的Rsapbian中，增加了很多壁纸，默认的桌面是一条公路，不如树莓派之前的Logo清爽，如果需要更换，右键鼠标——Desktop Preferences(桌面首选项)——Desktop选项卡中，可以选择图片，以及图片显示的方式。

在这里，我选择了树莓派的Logo（raspberry-pi-logo.png），居中显示(Centre image on screen),效果如下：

![logo设置][logo设置]

## 增加中文输入法
默认的Rsapbian系统中是不支持拼音输入的，需要手动安装一下，在Shell中输入以下指令，可以安装Fcitx框架以及谷歌拼音输入法（需要联网），安装成功后重启，右上角应该出现一个键盘托盘，点击可以切换英文输入法和中文输入法。
```
sudo apt-get install fcitx fcitx-googlepinyin
```

## 切换中文环境以及编码方式
对于英文不好的同学，树莓派也默认提供了中文环境的支持，同样是开始菜单（左上角）——Preferences（首选项）——Raspberry Configuration（树莓派配置）——Localisation（本地化）——SetLocal（设置地区）——language（语言），在这一项中选择最下面的ZH（chinese），然后默认编码方式也会变为uft-8，不会显示中文乱码，重启树莓派之后，操作系统的基本选项就变为中文了。

![中文设置][中文设置]

## 设置固定IP
由于树莓派携带过程中，有可能遇到没有WiFi的环境，如果此时需要连接，就显得比较麻烦，可以设置有线网卡的IP为静态IP，确保稳定连接，不会受到DHCP的影响，况且有时候即使IP分配正常，没有浏览器的情况下也不便于查看IP，arp指令有时无法找到树莓派。

![IP设置][IP设置]

右键点击WiFi连接——Wireless&Wired Network Setting(无线和有线网络设置)——选择网卡eth0，将IP地址设置为固定IP，比如说192.168.1.199，网关设置为192.168.1.1，这样将电脑和树莓派连接时，把电脑的IP设置为192.168.1.x，网关设置为192.168.1.1，就可以直接通信了，而无需通过无线网自动分配。

## 保存WiFi连接
在Raspbian中，连接过的WiFi会被自动保存到系统中，因此我们也可以手动修改该文件，对可连接的WiFi进行保存设置。

在SSH中，输入以下指令，打开WiFi配置文件wpa_supplicant.conf
```
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf    //使用nano编辑器打开配置文件
sudo vi /etc/wpa_supplicant/wpa_supplicant.conf      //使用vi编辑器打开配置文件
```

![WiFi设置][WiFi设置]

如果你之前连接过，则此文件下会出现如下配置
```
network={
        ssid="mywifi"             //ssid是WiFi广播名称
        psk="12345678"            //psk是连接密码
        key_mgmt=WPA-PSK          //key_mgmt是连接类型，一般不用修改
        priority=9                //priority是优先度，非必选项，会按照优先度连接
}
```
你只需要在该文件中增加这样一个语块即可保存WiFi了。

[修改密码]: /assets/pic/2017-08-07/changePassword.jpg
[logo设置]: /assets/pic/2017-08-07/desktopPreferences.jpg
[中文设置]: /assets/pic/2017-08-07/setlanguage.jpg
[IP设置]: /assets/pic/2017-08-07/setIP.jpg
[WiFi设置]: /assets/pic/2017-08-07/wpaconfig.jpg