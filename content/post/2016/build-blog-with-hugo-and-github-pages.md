+++
date = "2016-02-21T16:36:54+08:00"
description = "本文记录了我用Hugo和GitHub Pages搭建静态博客的过程，以及我对此的一点想法"
draft = false
tags = []
title = "用Hugo和GitHub Pages搭建博客"
topics = []

+++

从前天晚上开始尝试了一下用Hugo和GitHub Pages搭建静态博客，总体还算顺利（主要是Hugo用起来比较方便）。

**关于 Hugo**

Hugo 是由 spf13 创建的一个静态网站生成器，首页：https://www.gohugo.io/

# Hugo 工作流

下面将介绍Hugo的完整工作流程：

## 快速安装
在 https://www.gohugo.io/ 能够找到 Homebrew 安装 Hugo的命令，

以及Download： https://github.com/spf13/hugo/releases

下载对应的操作系统版本的Hugo二进制文件（hugo或者hugo.exe），添加到`$PATH`能找到的目录下。

<!--more-->

## 创建站点
```bash
$ hugo new mysite
```

会在mysite目录生成原始（未经渲染的）站点，如下：

```
$ cd mysite
$ tree mysite
.
├── archetypes
├── config.toml
├── content
├── data
├── layouts
└── static

```


## 创建文章
```bash
$ hugo new about.md
```
会在content下创建一个about.md。

## 安装主题
在 http://themes.gohugo.io/ 选号主题之后，将对应主题下载到 `themes/` 下，如hyde。

### 配置主题
不同主题会有不同可配置的参数，按照说明修改config.toml即可。

## 生成页面
```
$ hugo --theme=hyde
```
该命令会将所有md渲染成HTML。

### 本地预览
```
$ hugo server --theme=hyde --buildDrafts --watch
```
可在本地预览渲染效果。


## 发布
hugo命令默认生成html到 `public/` 下，可用 `-d` 选项渲染结果指向其他目录，
或者在执行hugo命令前将 创建软连接 public 指向其他目录。

GitHug Pages提供了静态网页和Jeklly生成器功能，可以在 `USERNAME.githug.io` 仓库
或其他仓库的`gh-pages`分支部署静态网页，或Jeklly原始页面。

# Hugo, Jeklly, Hexo 简单对比

在使用 Hugo 之前，我先后尝试了 Hexo、Jeklly， 说说感受。
Hexo的npm安装需要依赖很多其他node库，在网络条件不好的情况下，光安装就很费劲。
Jeklly也是，虽然相对Hexo来说依赖要少一些，但安装其他还是略显麻烦。

在经历过Hexo, Jeklly的痛苦之后，在安装Hugo的时候我直接选择了二进制安装。
这可能和实现技术有关。Hugo, Jeklly, Hexo分别是基于Go, Ruby, Node.js。
Go的编译型语言特性让Hugo**更容易**“二进制安装”。

所以相对来说，Hugo的主要优势在于安装方便，据说生成速度也更快。


# GitHub Pages 相对其他博客的好处
对于程序员来说，GitHub Pages相对于：
1. 注册类的博客网站，如CSDN
2. 自建独立博客，如WrodPress

GitHub Pages方式的好处在于————可以象管理代码一样管理博客，用git管理博客变更。

正如 [@ruanyf](https://github.com/ruanyf) 所说，“他们既拥有绝对管理权，
又享受github带来的便利----不管何时何地，只要向主机提交commit，就能发布新文章。
更妙的是，这一切还是免费的，github提供无限流量，世界各地都有理想的访问速度。”


# 参考链接
1. hugo中文文档 [gohugo.org](http://gohugo.org/)
2. http://blog.coderzh.com/2015/08/29/hugo/
3. http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html

