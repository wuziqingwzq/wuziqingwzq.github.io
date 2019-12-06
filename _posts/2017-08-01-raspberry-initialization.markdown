---
layout: post
title:  "树莓派学习（一）：系统安装"
date:   2017-08-01 18:45:00 +0800
categories: middleware
tags: raspberry raspbian
---
## 准备材料
树莓派安装系统，现在手上有以下物件：
1. 树莓派板子
2. USB电源5V 2A以上
3. Micro-USB电源线
4. SD卡
5. SD卡读卡器
6. HDML显示器
7. HDML线
8. USB鼠标
9. 无线路由器
10. 笔记本电脑（windows）

以下这些物品，如果有的话更好

- USB键盘
- 网线 

## 烧录系统
由于树莓派中没有存储部件，因此需要将系统安装到SD中

本文中安装的系统为官方推荐的raspbian

登陆官方网站[www.raspberrypi.org][www.raspberrypi.org],首页如下，点击下载（download），可以找到官方推荐的raspbian系统下载

![官方首页][官方首页]

在下载页面里面有两个download，到底下哪个呢？

![官方下载][官方下载]

- NOOBS     ：NOOBS是官方集成好的系统安装包，安装简便
- RASPBIAN  ：RSAPBIAN是系统镜像，需要用工具写入SD卡

本文中会讲解两个包的安装方法。

随便点击一个进去，都会提供两个下载地址，带LITE（右边）的是在线版系统，比较小，但是安装过程中需要联网下载，左边的是离线版。建议下载离线版的安装包，下载链接也是两个，左边下载种子，右边下载压缩包，有时候压缩包下载不了，可以用迅雷等软件使用种子下载。

![noobs下载][noobs下载]

![raspbian下载][raspbian下载]

### NOOBS安装

NOOBS的安装比较简单，用读卡器来读取SD卡，使用之前格式化一下，选择FAT32类型。

然后将解压缩的这些文件拷贝到SD卡中，注意是解压缩很多文件到根目录，不是一个文件夹。

![NOOBS解压缩][NOOBS解压缩]

然后把SD卡插到树莓派上（底部有个插槽，注意正反面）

接上HDML显示器，插上电源，不出问题的话，显示器会初始化并显示这个界面

![NOOBS安装界面][NOOBS安装界面]

里面有可以安装的系统选项，我们选择raspbian，系统就会自动开始安装了，这里还可以选择连接wifi，但是由于没有键盘，在下一步进行操作。

![NOOBS安装过程][NOOBS安装过程]

这个安装过程会持续不到半小时，完成之后就可以进入系统了。

![树莓派桌面][树莓派桌面]

### Rsapbian安装
Rsapbian系统安装也比较简单，在官网上下载的压缩包，打开后是一个.img的镜像文件，需要使用软件才能写入到SD卡中。

在网上下载一个Win32DiskImager，界面如下：

![Win32DiskImager][Win32DiskImager]

选择好你的镜像文件后，点击write写入，注意别点成read读取了，那是备份用的。

等待程序写入完成之后，将SD卡插入树莓派，上电就可以使用了。

[www.raspberrypi.org]: https://www.raspberrypi.org/
[官方首页]: /assets/pic/2017-08-01/raspbianorg.jpg
[官方下载]: /assets/pic/2017-08-01/raspbianorgDownload.jpg
[noobs下载]: /assets/pic/2017-08-01/NOOBSdownload.jpg
[raspbian下载]: /assets/pic/2017-08-01/raspbianDownload.jpg
[NOOBS解压缩]: /assets/pic/2017-08-01/NOOBSRAR.jpg
[NOOBS安装界面]: /assets/pic/2017-08-01/install.jpg
[NOOBS安装过程]: /assets/pic/2017-08-01/installing.jpg
[树莓派桌面]: /assets/pic/2017-08-01/raspbianDesktop.jpg
[Win32DiskImager]: /assets/pic/2017-08-01/Win32DiskImager.jpg