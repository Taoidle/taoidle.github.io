---
title: 关于Ubuntu20.04安装arm-linux-gcc发生的错误
tags:
  - ARM
  - arm-linux-gcc
  - gcc
  - ubuntu
id: '327'
categories:
  - - 折腾记录
date: 2020-06-12 01:30:58
---

ubuntu20.04安装arm-linux-gcc发生错误:libpng3 : Depends: libpng12-0 (>= 1.2.5.0-2) but it is not installed ![](https://cdn.site.taoidle.com/wp-content/uploads/2020/06/wp_editor_md_ffc711dadcd44f60ffc78cf327cc4886.jpg) 解决方法：

```shell
sudo add-apt-repository ppa:linuxuprising/libpng12

sudo apt update

sudo apt list --upgradable

sudo apt --fix-broken install

sudo apt install gcc-arm-linux-gnueabi

```