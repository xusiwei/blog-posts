+++
date = "2016-03-09T23:38:56+08:00"
description = ""
draft = false
tags = []
title = "NGINX的并发模型"
topics = []

+++

## Apache 的并发模型
Apache 的早期版本的并发模型是 one process per connection，也是早期Unix服务端的主流模型：
主进程accept阻塞在listen fd上，有新的HTTP请求到来时（accept返回connection fd），主进程fork一个服务进程，
专门为该连接服务，HTTP会话结束退出此进程。

Apache 后来的版本，改用one thread per connection，改用单独一个线程为客户端连接提供服务。
这个模式和之前版本的一样，listen fd和connection fd用的都是阻塞模式。

使用阻塞模式的好处是，逻辑清晰，编程模型简单。process per connection的好处还有可以保证客户端连接的稳定性
（据说早期Unix的某些版本上多进程模型并不稳定），同时也保证了安全性（即使客户端通过hack手段在connection上拿到了shell也只能控制服务进程）。

Tomcat 使用的并发模型也是 one thread per connection。开发环境下，在servlet或jsp代码中上打断点，
在有超过一个浏览器访问有断点的servlet时，能够看到Tomcat为每个连接创建了一个线程。

这种模型下，单个服务器的服务能力主要受到内存的限制，因为每创建一个进程/线程都要消耗一定数量的内存（MB级别）。

## NGINX 的并发模型
NIGNX 支持 single 模式 和 master + worker 模式。

### NGINX master + worker 模式

NGINX通常工作在master + worker模式下，master和worker都是单线程的进程，用户可以通过增加worker进程数实现提高多核CPU的利用率。
同时，这种每个进程仅有一个线程的模型，使得不需要时刻考虑数据的保护问题（如果是多线程则需要考虑）。

来自Andrew Alexeev的一张图：
![NGINX architecture](http://www.aosabook.org/images/nginx/architecture.png)

#### master 进程
NGINX的master进程，负责解析配置文件，并启动其他角色的进程，如启动worker，cache manager。

启动完成其他进程之后，master进程仅起监视作用（相当与monitor角色），后面用户可以通过singaller角色的nginx向master发命令，
master再通知它创建起来的其他进程。

#### worker 进程
worker进程是实际服务客户的进程。包括接受新连接（accept），相应连接上的消息（read/write）.

#### singaller 进程
singaller进程仅仅是一个向master进程发送信号的工具进程。
通过向master发生特定消息，能够让：
* master/worker完成当前服务后，正常退出。
* master进程重新加载配置文件，或者重启worker。
* 重新打开log 文件。


#### cache manager
可以做 HTTP、文件缓存，即对同一个静态文件的请求进行缓存。

<!--more-->

## more
[Chapter “nginx” in “The Architecture of Open Source Applications”](http://www.aosabook.org/en/nginx.html)
