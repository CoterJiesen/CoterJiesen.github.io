---
layout: post
title: Debian配置为VPN服务器
category: 技术
tags: CPP;Linux;VPS
keywords: 
description: 
---
#Debian配置为VPN服务器


今天接到一个任务，就是将一台Debian配置为VPN服务器。以前配置过windowsserver下的VPN服务，linux下的还真没有配置过，顺便也就练练手。其实也不是很难，安装网上一些资料坐下来就可以了。


第一步、检查环境做VPN的Debian需要把PPP和TUN启用
运行一下命令来检查ppp和tun环境
	#cat /dev/ppp
显示cat: /dev/ppp: No such device or address为正常。否则，尝试运行以下命令后再试一下（我的vps就是这种情况，导致废弃了一个月，今天才发现这方法！）

	root@debian01:~# rm /dev/ppp
	root@debian01:~# mknod /dev/ppp c 108 0

	root@debian01:~# cat /dev/net/tun
显示cat: /dev/net/tun: File descriptor in bad state为正常

如果顺利通过，那么很好，继续往下看吧，否则，就要去Debian的控制面板上看看有没有开关。

第二步、安装pptpd

	root@debian01:~# apt-get install pptpd
很快就会安装完毕的，只要你的Debian可以上网，如果不能上网没有安装apt-get可能安装就比较麻烦点。
在debian下如何查看有没有安装某个软件包，和Redhat及CentOS不一样哦，如果您不确定某个软件包是否已经安装，可以使用 dpkg 的 -l (L的小写) 选项：
$ dpkg -l xxx   /xxx是软件包的名称

注意：Debian下的vi编辑器和我们在Redhat下使用的不一样，其实Redhat下的vi全名是vim.而Debin下的vi和unix下的一样。第一次操作很不爽啊，上下移动不会，删除不会啊。果断的使用#apt-get install vim安装vim来使用。查了一下默认安装的vi的使用方法：
方向键
上k
下j
左h
右l
x   删除游标所在字元。

第三步、配置pptpd
root@debian01:~# vi /etc/pptpd.conf
去掉下面两行的注释

	#localip 192.168.0.1
	#remoteip 192.168.0.234-238,192.168.10.245
如果本机使用路由器之类的上网，而ip段又正好是192.168.0.xxx的话，那需要把上面的ip改成其他ip段，比如：
	
    localip 192.168.100.1
	remoteip 192.168.100.234-238


记得在配置之前，为了防止配置错误无法复原，需要备份一下配置文件，使用如下命令：

	root@debian01:~# cp /etc/pptpd.conf/etc/pptpd.conf.bak
修改前pptpd.conf的配置

这里为了不好其他局域网网段冲突，我这里改为了200.0/24网段。（此次服务搭建是在内网环境中做的，和公网环境其实一样，只是ip地址不同而已）
第四步、配置 DNS

	root@debian01:~# vi /etc/ppp/options
加入这两行（文件很长，直接在头部加入即可）

    ms-dns 8.8.8.8
    ms-dns 8.8.4.4
我的配置：
1dns的配置.jpg
第五步、开启IP转发
root@debian01:~# vi /etc/sysctl.conf
去掉net.ipv4.ip_forward=1的注释肉眼找不到可以输入/net.ipv4.ip_forward=1，按回车搜索。

第六步、添加用户名密码
root@debian01:~# vi /etc/ppp/chap-secrets
用户名 pptpd 密码 * 按照这样的格式输入（注意最后的*也要加上）

这里有有两行，前面一个是指定IP，第二个是不用指定客户端获取的IP地址，随机重地址段中分配。
第七步、编写转发规则

	root@debian01:~# vi /etc/pptpdfirewall.sh
这是新建的空白文件。写入一下内容

    /sbin/iptables -t nat -A POSTROUTING -s192.168.0.0/24 -j SNAT --to-source ***.***.***.***
    /sbin/iptables -A FORWARD -s 192.168.0.0/24-p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -j TCPMSS --set-mss 1356
其中的***.***.***.***替换成Debian的公网IP。如果是之前讲的ip段冲突的情况，要把两个192.168.0.0/24都改成192.168.100.0/24（用自定义IP的话，自己参照着写）

注意上图配置防火墙转发规则时，*号IP我换成了我的eth1的IP地址，如果是公网环境也就是公网静态IP地址。-s地址就是VPN服务器将分配给客户端的地址段。
然后设置文件为可执行

	root@debian01:~# chmod 755/etc/pptpdfirewall.sh
第八步、加入开机启动

	root@debian01:~# vi /etc/init.d/rc.local
在文件尾加上

	sh /etc/pptpdfirewall.sh

第九步、重启Debian

	root@debian01:~# reboot

注意，不要用网上所谓的给require-mppe-128加注释的方法。这个方法治标不治本，因为mppe是用于加密的，注释了等于没有加密，数据还是公开的，这样可能平时能访问一些网站，但是关键时刻和关键网站就无能为力了，因为数据照样能被过滤的。不加密的VPN一点用处都没有，还不如直接用SSH的好。

