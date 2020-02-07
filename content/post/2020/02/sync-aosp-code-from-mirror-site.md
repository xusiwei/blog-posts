---
title: "从开源镜像站下载Android系统源码"
date: "2020-02-07T20:42:51+08:00"
draft: false
toc: true
tags: []
categories: []
---

国内可以用来下载Andorid源码的开源镜像站有：

* [科大开源镜像站](https://mirrors.ustc.edu.cn/)
* [清华开源镜像站](https://mirrors.tuna.tsinghua.edu.cn/)

***Tips:***

1. 不同地区（或不同运营商）的网络，从这两个镜像站下载代码的速度会有所差别；
2. 清华AOSP镜像只支持HTTPS协议下载，科大AOSP镜像同时支持git协议和HTTPS协议下载；（git协议下载更快一些，如果网络允许的话，优先选择科大镜像）；
3. 可以通过`curl -I`发送`HEAD`请求，简单测试一下哪个站点更优（使用`time`命令计时，时间短的更优）：
    * `time curl -I https://mirrors.ustc.edu.cn/`
    * `time curl -I https://mirrors.tuna.tsinghua.edu.cn/`

科大镜像下载AOSP代码的推荐流程：

1. 下载配置repo和REPO_URL
2. 创建目录、`repo init`初始化仓库
3. `repo sync`同步代码

清华镜像站下载AOSP代码的推荐流程（也可以使用和科大镜像站类似的方法）：

1. 下载配置repo和REPO_URL
2. 下载月度备份的`aosp.tar`包初始化（下载完成后解压`aosp.tar`包）
3. 在月度备份包基础上`repo sync`

## 下载命令行工具——`repo`

***下载***:

```bash
mkdir ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
# 如果失败，从清华镜像站下载repo命令：
# curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo > ~/bin/repo
chmod a+x ~/bin/repo
```

也可以从科大镜像站下载`repo`命令：

```bash
curl -sSL  'https://gerrit-googlesource.proxy.ustclug.org/git-repo/+/master/repo?format=TEXT' |base64 -d > ~/bin/repo
```

***配置***:

在`～/.bashrc`里面添加：

```bash
# 将repo命令所在目录加到PATH环境变量
PATH=~/bin:$PATH

# repo sync 命令执行时，会首先使用该变量的远程仓更新本地仓`.repo/repo`
export REPO_URL = 'https://gerrit-googlesource.proxy.ustclug.org/git-repo'

# 也可以选择清华镜像站
# export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
```

REPO_URL可以设置为环境变量加到`.bashrc`里面，也可以修改`～/bin/repo`的代码：

```python
import os
REPO_URL = os.environ.get('REPO_URL', None)
if not REPO_URL:
  # REPO_URL = 'https://gerrit.googlesource.com/git-repo'
  REPO_URL = 'https://gerrit-googlesource.proxy.ustclug.org/git-repo'
```

## 从镜像站同步代码

### 从科大镜像站下载

```bash
# 创建工作目录
mkdir WORKING_DIRECTORY
cd WORKING_DIRECTORY

# 初始化仓库
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b master

# 同步源码
repo sync
```

### 从清华镜像站下载

```bash
# repo init方式（较慢）
mkdir WORKING_DIRECTORY # 创建工作目录
cd WORKING_DIRECTORY
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b master # 初始化仓库

# 下载tar包方式（下载月度备份包，较快）
# wget -c https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar # 下载初始化包
# tar xf aosp-latest.tar
# cd AOSP   # 解压得到的 AOSP 工程目录

# 同步源码
repo sync
```

## 参考

* [科大AOSP wiki](https://lug.ustc.edu.cn/wiki/mirrors/help/aosp)
* [清华AOSP wiki](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)
