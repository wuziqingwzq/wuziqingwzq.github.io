---
layout: post
title:  "使用注册表设置IE浏览器"
date:   2017-06-12 15:04:00 +0800
categories: other
tags:   IE reg
---
# 用途
对我来说，在浏览器中要使用数字证书，通常会使用到各种控件，各种弹窗与ActiveX拦截很不友好，需要在安装程序中设置信任站点等，修改这些注册表信息即可完成。

当然，需要批量给电脑修改IE浏览器设置的时候，直接写一个注册表文件也很方便。

## 注册表头信息
建立一个.reg文件，第一句需要加上这一句，否则系统不会识别为一个有效的注册表信息
```
Windows Registry Editor Version 5.00
```

## 信任站点设置
将某个网站直接加入信任站点，比如https://www.testpage.com

其中可以修改的东西有三项：
1. 注册表项中的ranges+数字，这个数字是自定义的，从ranges01开始，每增加一个信任站点就会加一一般来说避免冲突可以选一个大一点的数，比如我选择的500。
2. 域名或者IP，可以用通配符
3. 协议号，可以变成http或者https

```
[HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Ranges\Range500]
"https"=dword:00000002  //协议类型，可修改未http或ftp等
":Range"="www.testpage.com"   //域名或IP地址，可修改
```

## 信任站点安全设置
设置可信站点区：
```
[HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Zones\2]
@=""
"DisplayName"="可信站点"
"PMDisplayName"="Trusted sites [Protected Mode]"
"Description"="该区域中包含您确认不会损坏计算机或数据的网站。"
"Icon"="inetcpl.cpl#00004480"
"LowIcon"="inetcpl.cpl#005424"
"CurrentLevel"=dword:00000000
"Flags"=dword:00000047
"1001"=dword:00000000    //下载已签名的 ActiveX 控件
"1004"=dword:00000000    //下载未签名的 ActiveX 控件 
"1200"=dword:00000000    //运行 ActiveX 控件和插件
"1201"=dword:00000000    //对没有标记为安全的 ActiveX 控件进行初始化和脚本运行
"1405"=dword:00000000    //对标记为可安全执行脚本的 ActiveX 控件执行脚本
"2201"=dword:00000000    //ActiveX 控件自动提示
```

## 弹出窗口阻止程序
浏览器中有一项设置，是工具——弹出窗口阻止程序——启用弹出窗口阻止程序/停用弹出窗口阻止程序
```
[HKEY_CURRENT_USER\Software\Microsoft\Internet Explorer\New Windows]
"PopupMgr"="no"       //no为关闭，yes为开启
//弹窗管理设置为关闭
```

## 兼容性视图设置
兼容性视图的设置比较麻烦，由于网址列表会被编码成十六进制，所以不能直接将参数写进去。需要做如下操作：

1. 先打开浏览器，将需要设置的网址添加到兼容性视图中。
2. 打开注册表编辑器（WIN+R，输入regedit），找到注册表项``[HKEY_CURRENT_USER\Software\Microsoft\Internet Explorer\BrowserEmulation\ClearableListData]``,应该会有一项UserFilter，直接导出该项注册表。
3. 如下所示，通过该注册表可以批量添加之前设置的网址到兼容性视图。

```
[HKEY_CURRENT_USER\Software\Microsoft\Internet Explorer\BrowserEmulation\ClearableListData]
"UserFilter"=hex:41,1f,00,00,53,08,ad,ba,05,00,00,00,ea,00,00,00,01,00,00,00,\
  05,00,00,00,0c,00,00,00,1a,21,6a,f1,2e,e3,d2,01,01,00,00,00,0e,00,67,00,7a,\
  00,73,00,67,00,67,00,7a,00,79,00,6a,00,79,00,7a,00,78,00,2e,00,63,00,6e,00,\
  0c,00,00,00,c8,0c,b7,36,2f,e3,d2,01,01,00,00,00,0f,00,32,00,32,00,30,00,2e,\
  00,31,00,39,00,37,00,2e,00,32,00,31,00,39,00,2e,00,31,00,38,00,34,00,0c,00,\
  00,00,d4,0e,cd,3c,2f,e3,d2,01,01,00,00,00,0e,00,32,00,32,00,30,00,2e,00,31,\
  00,39,00,37,00,2e,00,31,00,39,00,38,00,2e,00,38,00,39,00,0c,00,00,00,38,47,\
  1d,56,2f,e3,d2,01,01,00,00,00,0d,00,61,00,6e,00,73,00,68,00,75,00,6e,00,2e,\
  00,67,00,6f,00,76,00,2e,00,63,00,6e,00,0c,00,00,00,a0,4c,92,68,2f,e3,d2,01,\
  01,00,00,00,0a,00,6c,00,70,00,73,00,67,00,67,00,7a,00,79,00,2e,00,63,00,6e,\
  00
```
