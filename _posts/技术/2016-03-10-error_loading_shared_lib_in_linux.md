---
layout: post
title: error while loading shared libraries的解決方法
category: 技术
tags: CPP Linux VPS
keywords: 
description: 
---

##error while loading shared libraries的解決方法


在linux下运行程序时，发现了error while loading shared libraries这种错误，一时间不知道解决办法，在网上搜索，终于解决了：

 

./tests: error while loading shared libraries: xxx.so.0:cannot open shared object file: No such file or directory
出现这类错误表示，系统不知道xxx.so放在哪个目录下，这时候就要在/etc/ld.so.conf中加入xxx.so所在的目录。

一般而言，有很多的so会存放在/usr/local/lib这个目录底下，去这个目录底下找，果然发现自己所需要的.so文件。

所以，在/etc/ld.so.conf中加入/usr/local/lib这一行，保存之后，再运行：/sbin/ldconfig –v更新一下配置即可。