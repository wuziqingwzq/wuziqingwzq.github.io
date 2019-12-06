---
layout: post
title:  "修改Oracle管理员密码"
date:   2017-06-28 00:00:00 +0800
categories: middleware 
tags: oracle password sqlplus
---

# 解决的问题
今天在项目中需要用到很久没打开过的数据库，但是在登陆时发现账户名和密码都忘了，一直提示
```
ORA-01017: invalid username/password; logon denied
```
所以记录一下今天处理的方法，顺便帮助遇到同样问题的人。

1. 修改Oracle管理员密码
2. 修改需要使用的用户密码

***
# 方法

## 管理员身份进入数据库

需要以非登陆模式进入SQL界面

win+R键，输入CMD，进入命令提示符，输入如下语句：

```
sqlplus /nolog
```
弹出以下界面就算成功了

```
SQL*Plus: Release 11.2.0.1.0 Production on 星期三 6月 28 17:38:20 2017

Copyright (c) 1982, 2010, Oracle.  All rights reserved.

SQL>
```

接下来使用以下语句以数据库管理员sysdba的身份进入数据库

```
conn /as sysdba
```

有时会出现这样的错误
```
ERROR:
ORA-01031: insufficient privileges
```

## 管理员登陆不上
有可能是当前登陆的账户没有权限作为数据库管理员，需要检查一下

在Win+R后输入compmgmt.msc（也可以右键计算机，点击管理），进入计算机管理computerManagement界面。

选择**本地用户和组**——**组**

![计算机管理][计算机管理]

然后在列表中找到名为**ora_dba**的组，看看里面有没有当前登陆的windows用户。

![用户组][用户组]

如果没有的话，添加一下。然后切换出去再试试是否可以使用管理员登陆。

```
SQL> conn /as sysdba
已连接。
```
提示这个则说明已经管理员身份进入系统。

## 修改用户以及密码

使用以下指令,将用户名为**yourUsername**的账户密码改为**yourPassword**。
```
alter user yourUsername identified by yourPassword
```









[计算机管理]: /assets/pic/2017-06-28/compmgmt.png
[用户组]: /assets/pic/2017-06-28/sysdbanull.png