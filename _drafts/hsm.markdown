---
layout: post
title:  Docker基本命令
date:   2019-12-19 09:00:00 +0800
categories: middleware
tags: Docker
---


## Docker

docker安装
docker加速

显示当前镜像的列表
sudo docker images
显示当前容器的列表及状态
sudo docker ps -a
创建一个名为test的Ubuntu容器，关联到/bin/bash
sudo docker run --name test -i -t ubuntu /bin/bash 
创建一个名为test的Ubuntu容器，在后台以进程方式运行
sudo docker run --name test -d ubuntu /bin/bash
查看top容器中的进程
sudo docker top test
