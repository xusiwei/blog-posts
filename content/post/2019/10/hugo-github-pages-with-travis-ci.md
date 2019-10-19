+++
date = "2019-10-18T22:58:33+08:00"
description = ""
draft = false
tags = ["CI/CD"]
title = "用Travis CI自动化Hugo生成及GitHub Pages部署"
categories = ["操作指南"]
+++

# Automation Hugo Generation and GitHub Pages Deployment with Travis CI

## 背景

使用Hugo作为个人博客生成器有一段时间了，博客原始文稿的markdown也是用了GitHub仓库管理。所以每次写完东西之后，除了markdown原始文稿需要提交一次，生成的静态站点内容也需要提交一次，有点麻烦。

## 尝试

因为之前有用过免费的TravisCI，一直想着能不能通过它实现只提交一次原始Markdown文稿，自动生成一下静态站点、并上传到GitHub Pages仓。于是初步尝试了一下，验证了这样是可行的:

```YAML
language: c

os:
  - linux

before_install:
  - wget --quiet https://github.com/gohugoio/hugo/releases/download/v0.58.3/hugo_0.58.3_Linux-64bit.tar.gz
  - tar xzvf hugo_0.58.3_Linux-64bit.tar.gz
  - export PATH=$PATH:$PWD

before_script:
  - mkdir public && git clone https://github.com/$GITHUB_ID/$GITHUB_ID.github.io.git public

script:
  - hugo

after_success:
  - echo Generate success, deploy site to GitHub Pages.
  - subject=`git log --pretty=%s -n 1`
  - cd public
  - git add .
  - git -c user.name='travis-ci' -c user.email="$GITHUB_EMAIL" commit -m "$subject"
  - git push -f https://$GITHUB_ID:$GITHUB_PWD@github.com/$GITHUB_ID/$GITHUB_ID.github.io.git

after_failure:
  - echo build failure
```

其中的三个变量分别是：

* GITHUB_ID 保存GitHub用户名
* GITHUB_PWD 保存GitHub密码
* GITHUB_EMAIL 保存GitHub账号的一个邮箱

这三个变量需使用Travis环境变量的方式设置值，并且注意创建时**不要打开**"DISPLAY VALUE IN BUILD LOG"选项。否则，Travis构建的日志中就能够看到你的这几个变量的值了：
![add env vars](../add_env_var.png)

虽然这样确实可以实现自动生成和部署，但是总感觉直接使用密码的方式不太安全。

试完之后搜了一下"TravisCi + Hugo + Github Pages"，
从别人的博客里发现，原来TravisCI自带了GitHub Pages的部署支持。
TravisCI官方GitHub Pages部署文档链接： https://docs.travis-ci.com/user/deployment/pages/

于是我把我的`.travis-ci.yml`文件的部署方式也改成使用这种方式：

```YAML
language: c

os:
  - linux

install:
  - wget --quiet https://github.com/gohugoio/hugo/releases/download/v0.58.3/hugo_0.58.3_Linux-64bit.tar.gz
  - tar xzvf hugo_0.58.3_Linux-64bit.tar.gz
  - export PATH=$PATH:$PWD

script:
  - hugo

deploy:
  repo: $GITHUB_USER_ID/$GITHUB_USER_ID.github.io # 部署GitHub的仓，不需要 https://github.com/ 前缀
  provider: pages # 重要，指定这是一份github pages的部署配置
  skip-cleanup: true # 重要，不能省略
  local-dir: public # 重要，静态站点文件所在目录
  target-branch: master # 重要，要将静态站点文件推送到哪个分支
  github-token: $GITHUB_API_TOKEN # 重要，需要在GitHub上申请、再到配置到Travis
  keep-history: true # 是否保持target-branch分支的提交记录
  on:
    branch: master # 博客源码拉取的分支
```


参考： https://www.metachris.com/2017/04/continuous-deployment-hugo---travis-ci--github-pages/


