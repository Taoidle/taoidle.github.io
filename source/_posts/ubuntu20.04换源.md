---
layout: ubuntu20.04换源
title: ubuntu20.04换源
tags:
  - Linux
  - ubuntu
  - ubuntu20.04
categories:
  - - 折腾记录
date: 2020-06-12 01:52:27
---

依次执行以下命令

```shell
sudo apt install vim        安装vim编辑器

sudo  cp   /etc/apt/sources.list   /etc/apt/sources.list.bak        备份源文件

sudo  chmod  777  /etc/apt/sources.list         修改权限

sudo  vim   /etc/apt/sources.list       编辑源文件
```

在vim编辑器中将原来内容替换为

```shell
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security multiverse
```

这里为清华源，可将链接替换为你喜欢的软件源 然后执行

```shell
sudo apt list --upgradable 

sudo apt update

sudo apt upgrade
```

结束语：啊，又水了一篇博文，快乐（不是