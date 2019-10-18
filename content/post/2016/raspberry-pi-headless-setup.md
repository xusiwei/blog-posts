+++
date = "2016-03-26T09:31:01+08:00"
description = ""
draft = false
tags = ["RaspberryPi"]
title = "Raspberry Pi 3 无显示器 安装指南"
categories = ["开源硬件"]

+++

本文记录了——在没有显示器、键盘的条件下，如何通过网线为树梅派（后面简称pi）安装系统。

实验环境如下图。

![raspberrypi-rj45-laptop](/post/2016/raspi-rj45-laptop.png)


## 下载系统镜像
官网的[下载页面](https://www.raspberrypi.org/downloads)可以找到pi的系统镜像。

官方首推新手使用的是NOOBS和raspbian，而NOOBS和Raspbian都提供了完整版和Lite版。

NOOBS的完整版是包含了Raspbian，可以离线安装raspbian。Lite版仅有一个系统安装程序，需要连接网络才能安装具体的操作系统。

Raspbian的完整版是一个完整的桌面镜像，Lite版预装的软件包要少一些（联网后可以继续安装）。

我下载的是[RASPBIAN JESSIE](http://director.downloads.raspberrypi.org/raspbian/images/raspbian-2016-03-18/2016-03-18-raspbian-jessie.zip)

下载完成后解压zip文件，将会得到一个img文件。

<!--more-->

## 将镜像写入SD卡

Windows系统上可以用 [Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/) 进行写入。

Linux/Mac OS上可以用dd命令。

插入sd卡之前可以看看系统当前有哪些存储设备：
```
ls /dev/sd*
```

插入sd卡之后，再看一次，前后比较，就可以知道sd卡对于的设备路径了。


比如你的sd卡设备是`/dev/sdx`，可以用如下命令，将镜像写入sd卡。
```
dd bs=4M if=2016-03-18-raspbian-jessie.img of=/dev/sdx
```
写入镜像的过程比较慢，而且dd命令没有输出，可以在另一个终端里用命令`sudo pkill -USR1 -n -x dd`向dd发信号，让dd报告当前进度。

## 启动pi

pi镜像文件写入sd卡完成后，就可以用sd卡启动pi了。

但为了能够通过网线连接pi，还需要再做一点配置。


### 修改pi的启动参数

下面介绍如何为网线的两端配置IP。

在将sd卡装到pi上之前，可以通过修改启动参数，为pi预分配一个IP地址。

具体做法是，查看刚写好的sd卡，找到 cmdline.txt，在最末尾加上：
```
ip=192.168.1.2
```
（PS: 需要注意的是，不能换行，`ip`是小写。）
这样，pi上电、启动后，会为它的网口分配`192.168.1.2`。


### 连接pi和PC

修改完pi的启动参数，就可以：

1. 将读卡器从PC上取下, 将sd卡装到pi上
2. 用网线将pi的网口和PC的网口连接起来
3. 插上pi的电源

### 修改PC的网口IP

接下来要做的是修改PC端的网口IP。Windows系统可以在“TCP/IP选项”设置，[参考此链接](http://www.howtogeek.com/howto/19249/how-to-assign-a-static-ip-address-in-xp-vista-or-windows-7/)。
```
xu@debian:~$ ifconfig eth0 | grep inet
xu@debian:~$ sudo ifconfig eth0 192.168.1.4
xu@debian:~$ ifconfig eth0 | grep inet
          inet addr:192.168.1.4  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::226a:8aff:fe32:f22c/64 Scope:Link
```

修改成功后，可以通过ping命令测试一下，是否两端的IP都已经按照预期配置了。
```
xu@debian:~$ ping 192.168.1.2
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=64 time=0.585 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=64 time=0.474 ms
64 bytes from 192.168.1.2: icmp_seq=3 ttl=64 time=0.468 ms
^C
--- 192.168.1.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.468/0.509/0.585/0.053 ms
```


## 使用 ssh 客户端连接到 pi

接下来就可以用ssh客户端链接pi了。Windows系统可以使用[PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)。
```
xu@debian:~$ ssh pi@192.168.1.2
```

pi的默认用户名、密码分别是`pi`、`raspberry`

密码输入正确后，可以看到：
```

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Mar 18 08:58:31 2016
pi@raspberrypi:~ $
```

### 展开文件系统

成功登录pi后，运行 raspi-config， 通过上下键选择，回车确认，TAB跳转。

选择第一项，“Expand Filesystem”。

至此，系统安装完成。


## 问题排除

整个过程中最重要一步是，为网线的两端分配IP。你可通过：

* ifconfig 查看PC的网口是否正常分配了IP
* ping 命令测试pi的网口是否正常分配了IP

整个过程中修改的文件仅有一个 cmdline.txt，如果pi端没有正常分配IP。从pi上取下sd卡，连到PC上，查看cmdline.txt是否按照正确的方式修改了。

如果pi还没能正常的IP，重复前面的步骤。

注意，PC上网线连接中断会清除IP。而pi的IP是通过启动参数配置的，所以不会轻易消失; 如果你需要让pi连接路由器，并自动获取IP，需要将启动参数上的ip删除。

## 引用链接

我PC的系统是Linux，本文记录的步骤仅对Linux系统用户有参考意义。

Windows用户参考：[此链接](http://www.circuitbasics.com/raspberry-pi-basics-setup-without-monitor-keyboard-headless-mode/)。

Mac用户参考：[此链接](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md)。

https://www.raspberrypi.org/documentation/installation/installing-images/linux.md
https://www.raspberrypi.org/blog/use-your-desktop-or-laptop-screen-and-keyboard-with-your-pi/
https://pihw.wordpress.com/guides/direct-network-connection/
