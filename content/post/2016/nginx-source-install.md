+++
date = "2016-03-07T23:35:28+08:00"
description = ""
draft = false
tags = ["Nginx"]
title = "NGINX 源码安装"
categories = ["使用指南"]

+++

NIGNX 是由 Igor Sysoev 开发的一个 Web 服务器。
官网：http://nginx.org/


下载页面： http://nginx.org/en/download.html


## 下载 NGINX

以 1.8.1 版本为例：

```
wget http://nginx.org/download/nginx-1.8.1.tar.gz
```


## 下载依赖库

NGINX必须依赖zlib和pcre, (若想让nginx支持HTTPS协议，还需OpenSSL)

NGINX 使用zlib进行gzip编/解码（压缩/解压缩）， zlib: http://zlib.net/

下载 zlib-1.2.8:

```
wget http://zlib.net/zlib-1.2.8.tar.gz
```


NGINX 使用pcre作正则表达式匹配， pcre: http://pcre.org/

ftp: ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/

下载 pcre-8.38:

```
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.38.tar.gz
```


## 解压

```
tar xzvf nginx-1.8.1.tar.gz

tar xzvf zlib-1.2.8.tar.gz

tar xzvf pcre-8.38.tar.gz
```


## 编译

```
cd nginx-1.8.1/

./configure --with-pcre=../pcre-8.38 --with-zlib=../zlib-1.2.8
```

## 安装

```
sudo make install
```

<!--more-->

默认将会安装到 `/usr/local/nginx` 目录
```
xusiwei@github:/usr/local/nginx$ tree
.
├── conf
│   ├── fastcgi.conf
│   ├── fastcgi.conf.default
│   ├── fastcgi_params
│   ├── fastcgi_params.default
│   ├── koi-utf
│   ├── koi-win
│   ├── mime.types
│   ├── mime.types.default
│   ├── nginx.conf
│   ├── nginx.conf.default
│   ├── scgi_params
│   ├── scgi_params.default
│   ├── uwsgi_params
│   ├── uwsgi_params.default
│   └── win-utf
├── html
│   ├── 50x.html
│   └── index.html
├── logs
└── sbin
    └── nginx
```

## 启动

源码编译安装生成的可执行文件位于 `/usr/local/nginx/sbin/`目录下，所以还需要将该目录加到`$PATH`下，
或者在`$PATH`的某个目录中创建一个符号链接指向 `/usr/local/nginx/sbin/nginx`

```
xusiwei@github:~$ cd /usr/local/bin/
xusiwei@github:/usr/local/bin$ sudo ln -s /usr/local/nginx/sbin/nginx nginx
xusiwei@github:/usr/local/bin$ nginx -v
nginx version: nginx/1.8.1
```

然后，就可以启动了：
```
xusiwei@github:~$ sudo nginx
xusiwei@github:~$
xusiwei@github:~$ ps -ef | grep nginx
xu       26065 24955  2 3月07 pts/2   00:00:53 kwrite content/post/2016/nginx-source-warmup.md
root     30077     1  0 00:08 ?        00:00:00 nginx: master process nginx
nobody   30078 30077  0 00:08 ?        00:00:00 nginx: worker process
xu       30080 17554  0 00:08 pts/1    00:00:00 grep nginx
```
不加参数启动，将会使用默认配置文件： `/usr/local/nginx/conf/nginx.conf`，并以 `/usr/local/nginx/` 为工作目录


## 测试

此时可以使用浏览器 http://localhost 或 http://127.0.0.1 看到效果了

如果是server版系统，可以用curl测试：
```
xusiwei@github:~$ curl http://localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```


## 停止

粗暴的方式，可以直接kill掉 nginx：
	* 先用 `ps -ef | grep nginx` 找到nginx的pid，再用 `kill pid`杀掉nginx
	* killall nginx
nginx -s参数提供了优雅的停止nginx的操作：
```
sudo nginx -s stop
```
初次之外， -s 还有 quit, reopen, reload


## 关于软件源安装

多数发行版的 软件源 中都已经收录了nginx，但通常版本都比较;
另外，如果你的发行版所属的发行商已经不再维护那个版本了，比如ubuntu 10.04，
所以，不推荐从默认软件源安装。

但如果觉得自己编译麻烦，可以从nginx.org的“源”安装预先编译好的软件包，具体步骤参见：http://nginx.org/en/linux_packages.html
