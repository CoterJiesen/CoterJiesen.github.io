---
layout: post
title: Centos查看端口占用情况和开启端口命令
category: 技术
tags: CPP;Linux
keywords: 
description: 
---

##Centos查看端口占用情况和开启端口命令

Centos查看端口占用情况命令，比如查看80端口占用情况使用如下命令：
 
	lsof -i tcp:80
 
列出所有端口
 
	netstat -ntlp
 
1、开启端口（以80端口为例）
 
      方法一：
 
         /sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT   写入修改
 
         /etc/init.d/iptables save   保存修改
 
        service iptables restart    重启防火墙，修改生效
 
       方法二：
 
       vi /etc/sysconfig/iptables  打开配置文件加入如下语句:
 
       -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT   重启防火墙，修改完成
 
 
 
2、关闭端口
 
     方法一：
 
         /sbin/iptables -I INPUT -p tcp --dport 80 -j DROP   写入修改
 
         /etc/init.d/iptables save   保存修改
 
        service iptables restart    重启防火墙，修改生效
 
       方法二：
 
       vi /etc/sysconfig/iptables  打开配置文件加入如下语句:
 
       -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j DROP   重启防火墙，修改完成
 
 
 
3、查看端口状态
 
      /etc/init.d/iptables status