+++
date = "2016-03-26T18:40:13+08:00"
description = ""
draft = false
tags = ["Raspberry Pi"]
title = "Raspberry Pi 使用科大镜像源"
topics = []

+++

[大开源镜像站](http://mirrors.ustc.edu.cn/)提供了raspbian的软件包镜像，国内的用户可以选择改用科大开源镜像站作为更新源。

科大镜像站官方已经提供了一份[帮助文档](https://lug.ustc.edu.cn/wiki/mirrors/help/raspbian)，用于指导用户如何使用科大镜像更新raspbian。
但这份文档是基于 wheezy 的，最新的jessie版可以参考本文。

## 具体做法
修改之前，最好先备份原始的配置文件。例如，使用如下命令将两个源配置文件拷贝到HOME目录。
```
cp /etc/apt/sources.list ~
cp /etc/apt/sources.d/raspi.list ~
```
或者将原有的配置注释掉。

### 修改 source.list
更新后的`/etc/apt/sources.list`：
```
#deb http://mirrordirector.raspbian.org/raspbian/ jessie main contrib non-free rpi
# Uncomment line below then 'apt-get update' to enable 'apt-get source'
#deb-src http://archive.raspbian.org/raspbian/ jessie main contrib non-free rpi

# use ustc mirror:
deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ jessie main contrib non-free rpi
```
实际修改是将`mirrordirector.raspbian.org`替换为`mirrors.ustc.edu.cn/raspbian/`。

<!--more-->


### 修改 raspi.list

更新后的`/etc/apt/raspi.list`：
```
#deb http://archive.raspberrypi.org/debian/ jessie main ui
# Uncomment line below then 'apt-get update' to enable 'apt-get source'
#deb-src http://archive.raspberrypi.org/debian/ jessie main ui

# use ustc mirror:
deb http://mirrors.ustc.edu.cn/archive.raspberrypi.org/debian/ jessie main ui
```
这里的修改是，将`archive.raspberrypi.org`替换为`mirrors.ustc.edu.cn/archive.raspberrypi.org`。
