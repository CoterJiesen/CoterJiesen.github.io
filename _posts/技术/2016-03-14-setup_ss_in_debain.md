---
layout: post
title: 在debian系统的vps上搭建shadowsocks-libev
category: 技术
tags: CPP Linux
keywords: 
description: 
---

##在debian系统的vps上搭建shadowsocks-libev

什么是shadowsocks

shadowsocks 是一个轻量级隧道代理，用来穿过防火墙。
目前服务端主流的是

    shadowsocks-nodejs
    shadowsocks-libev
    shadowsocks-Python
    shadowsocks-go
但是在小内存的vps还是要尽量节省内存，所以推荐shadowsocks-libev。
shadowsocks-libev有如下特点：
其一：版本更新及时，新功能支持的较多。比如目前最新的1.4.1版就支持UDP、多端口等最新功能。其二：资源占用极少，内存、CPU占用都非常低，即使是最低档64M内存的VPS都照跑不误。其三：部署、调试非常简单直观。

apt方式安装shadowsocks

小内存自然要使用debian，下面介绍在debian系统中使用apt方式搭建shadowsocks。

1、追加软件源

debian 6系统执行以下命令

	echo "deb http://shadowsocks.org/debian squeeze main" >> /etc/apt/sources.list
debian 7系统执行以下命令

	echo "deb http://shadowsocks.org/debian wheezy main" >> /etc/apt/sources.list
然后更新源并安装

	apt-get update
	apt-get install shadowsocks
2、配置shadowsocks

	vi /etc/shadowsocks/config.json
    {
              "server":"vps的ip",
              "server_port":8388,   #服务器端口，与SSH端口不一样
              "local_port":1080,
              "password":"barfoo!", #认证密码
              "timeout":60,
              "method":"aes-256-cfb" #加密方式，推荐使用aes-256-cfb
    }
3、重启shadowsocks服务。

    /etc/init.d/shadowsocks stop
    /etc/init.d/shadowsocks start
编译方式安装shadowsocks

编译安装方式版本一般比较新。

如果你已经采用apt方式安装过shadowsocks，那么使用如下命令删除已安装的shadowsocks

	dpkg -P shadowsocks
1、编译安装shadowsocks

安装编译所需包


    apt-get update
    apt-get install build-essential autoconf libtool libssl-dev git
然后通过git下载源码：

	git clone https://github.com/madeye/shadowsocks-libev.git
进入目录编译安装

    cd shadowsocks-libev
    ./configure --prefix=/usr
    make && make install
配置服务及配置文件

    mkdir -p /etc/shadowsocks-libev
    cp ./debian/shadowsocks-libev.init /etc/init.d/shadowsocks-libev
    cp ./debian/shadowsocks-libev.default /etc/default/shadowsocks-libev
    cp ./debian/config.json /etc/shadowsocks-libev/config.json
    chmod +x /etc/init.d/shadowsocks-libev
2、配置shadowsocks配置文件


	vi /etc/shadowsocks-libev/config.json

    {
              "server":"vps的ip",
              "server_port":8388,   #服务器端口，与SSH端口不一样
              "local_port":1080,
              "password":"barfoo!", #认证密码
              "timeout":60,
              "method":"aes-256-cfb" #加密方式，推荐使用aes-256-cfb
    }
3、重启shadowsocks服务。

    /etc/init.d/shadowsocks-libev stop
    /etc/init.d/shadowsocks-libev start
使用shadowsocks

关于在windows下及手机终端下如何使用，请参考：多平台使用Shadowsocks客户端

如果对vps不是太熟悉，可以参考：自己动手搭建Shadowsocks服务器
如果vps不是ovz环境的话可以参看该文章进行优化：Shadowsocks服务器TCP优化配置
在ios及安卓系统上使用可以参考该文章：多平台使用Shadowsocks客户端