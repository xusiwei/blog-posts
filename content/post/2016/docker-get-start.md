+++
date = "2016-03-08T22:58:33+08:00"
description = ""
draft = false
tags = ["Docker"]
title = "docker 入门指南"
categories = ["操作指南"]

+++

本文是实践官方“Get Started”的一个记录，由于我的桌面系统是Debian 8，本文的操作仅确保在 Debian 8 上正确。
指南链接（Linux版）： https://docs.docker.com/linux/


## 安装 Docker

### 官方脚本安装

如果没有 curl ，先安装 curl
```
sudo apt-get update
sudo apt-get install curl
```

用curl下载安装脚本并运行：
```
curl -fsSL https://get.docker.com/ | sh
```
安装成功后会提示你，使用如下命令：
```
sudo usermod -aG docker xu
```
将当前用户（xu）加到docker组，此后运行docker可以不加sudo。另外，让当前用户的组别改变需要重新登录（先log out再log in）。

### 验证

### bash
```
docker run -i -t debian docker
```
可以用docker创建一个交互的bash会话，在新的会话中运行ps -ef：
```
xu@debian:~$ docker run -i -t debian bash
root@d99cc11447c7:/#
root@d99cc11447c7:/#
root@d99cc11447c7:/# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  2 15:32 ?        00:00:00 bash
root         8     1  0 15:32 ?        00:00:00 ps -ef
```
可以看到，仅有两个进程，而且pid都非常小。这说明，这个会话环境已经与外界隔离了。

<!--more-->

#### hello world
安装成功后，验证：
```
user@host$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
535020c3e8ad: Pull complete
af340544ed62: Pull complete
Digest: sha256:a68868bfe696c00866942e8f5ca39e3e31b79c1e50feaee4ce5e28df2f051d5c
Status: Downloaded newer image for hello-world:latest

Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker Engine CLI client contacted the Docker Engine daemon.
 2. The Docker Engine daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker Engine daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker Engine daemon streamed that output to the Docker Engine CLI client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/userguide/
```
其中，`Hello from Docker.`及以后的内容是docker运行的结果，前面是镜像下载的log。


### 软件源安装
```
sudo apt-get install docker-engine
```
通常版本较低（不推荐）。

## 关于image和container
刚才的一个`docker run hello-world`命令，包含了docker引擎的主要工作，命令的三部分：
![docker command structure](/post/2016/docker-cmd-struct.png)

`container`（容器）是一个最精简的Linux操作系统。`image`（镜像）是可以装载到`container`的软件。
敲下hello-world命令后，docker引擎：
* 检查本地是否已经有`hello-world`镜像
* 如果没有，从Docker Hub下载该镜像
* 将镜像装入容器并运行它

`hello-world`进行输出一些信息后就退出了。一般情况下，镜像的具体行为和它的构建过程有关。

Docker镜像的功能远不至于此，它可以是像数据库那样的复杂软件，运行后等待你（或其他人）向其中添加数据。

Docker引擎让创建并分享软件变得简单——Docker容器总能保证镜像能够运行。


## 寻找、运行别人创建的镜像 —— Docker Hub
Docker Hub和GitHub类似，只是Docker Hub上托管的是Docker镜像而不是源码，这里你同样可给关注的镜像star。

在Docker Hub上能找到来自世界各地的人创建的docker镜像，docker hub的explore和GitHub的explore也非常类似，可在浏览其他人都创建了那些镜像，也能够看到各个镜像有多少star。

官方的`Get Started`里面以whalesay为例演示了如何用Docker Hub：
1. 在Docker Hub上搜索whalesay镜像
2. 运行whalesay镜像
```
$ docker run docker/whalesay cowsay boo
_____
< boo >
-----
   \
    \
     \
                   ##        .
             ## ## ##       ==
          ## ## ## ##      ===
      /""""""""""""""""___/ ===
 ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
      \______ o          __/
       \    \        __/
         \____\______/
```
whalesay是一个基于ubuntu 14.04构建的、命令行下输出字符图像的镜像，just for fun。
