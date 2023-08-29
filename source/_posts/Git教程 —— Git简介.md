---
layout: Git教程 —— Git简介
title: Git教程 —— Git简介
tags:
  - git
  - 教程
id: '79'
categories:
  - - Git教程
date: 2020-02-24 15:54:12
---

## **一、Git简介**

**1、Git是什么** Git(读音为/gɪt/。)是一个开源的分布式版本控制系统，可以有效、高速地处理从很小到非常大的项目版本管理。 \[1\] Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。（百度百科） 

**（1）集中式版本控制系统** 集中式版本控制系统的版本库集中存放在中央服务器的，干活的时候首先从中央服务器中获取最新的版本，然后干完活后在推送给中央服务器，本地没有版本库的修改记录。所以集中式版本控制器在没有联网的情况下是不能工作的。 

![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/1587996764-图片1.png) 

**（2）分布式版本控制系统** 分布式版本控制系统中没有中央服务器，在自己的电脑上是有版本库的，在你工作时不需要联网。但是分布式版本控制系统中需要有一台电脑充当中央服务器，以便于不同电脑修改的交换。 

![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/1587996849-图片2.png) 

**2、为什么要使用版本控制系统？** 编写文档、代码等文件的你一定有过这样的经历，对每次重要的修改都保存了一份新的文件 

![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/1587997204-图片3.png) 

这种做法相当简陋而且缺点非常明显，如果是多人协作的时候文档的命名不一定能反映出修改的内容，而且对存储空间的利用相当的不合理。这对生产需求来说是相当致命的，所以我们有了版本控制管理器。 版本控制器记录每次文件的改动，使用起来类似于如下效果

版本

文件名

用户

修改日期

说明

1

linux.doc

老王

2020.1.5 8.54

linux教程最初版

2

linux.doc

老王

2020.1.6 14.43

增加了文件权限相关内容

3

linux.doc

张三

2020.1.7 11.27

删除了硬件基础中老旧的硬件型号

4

linux.doc

李四

2020.1.8 18.36

对硬件基础中主板相关部分增加了新型号

5

linux.doc

老王

2020.1.9 16.09

增加了常用命令内容

这样你既可以方便的管理一个文件的多个版本，也可以跟同事协同工作，是不是非常方便？于是你开始使用Git进行版本管理，开启了新世界的大门。 

**3、安装Git** 

**（1）Linux安装Git**

```shell
Debian/Ubuntu: sudo apt-get install git

Fedora/Red Hat/CentOS: sudo yum install git-core
```

其它版本linux 从Git官网下载源码后解压，然后通过 ./config , make , sudo make install 安装，这里不在赘述。 

**(2) Windows安装Git** 

从[Git官网](https://git-scm.com/)下载安装包进行安装，安装之后找到Git Bash　打开后弹出命令行窗口即安装成功。 

**(3)MacOS安装Git** 

1、安装homebrew，brew安装Git。下边给出安装命令，在终端中输入

```shell
homebrew：/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

git: brew install git
```

2、在APP Store中下载Xcode，Xcode自带git 在完成安装之后我们要对git进行进一步的设置，打开Git Bash，在命令行中输入以下命令：

```shell
git config --global user.name “username” username为你的用户名

git config --global user.email “email@example.com” email@example.com为你的邮箱地址
```

注意：global参数是全局设置，表示该电脑上所有的git仓库都使用此用户名和邮箱地址