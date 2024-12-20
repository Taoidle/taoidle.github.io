---
layout: WSL镜像复制 & 常用命令
title: WSL镜像复制 & 常用命令
categories:
  - - 折腾记录
tags: 
  - wsl
date: 2024-12-20 16:21:42
---

# 场景

需要多开同个系统，比如开两个Debian

# 流程

- 注：以下以 Debian 为例

## 发行版设置

```shell

# 停止 Debian
wsl -t Debain
wsl --terminate Debian

# 取消注册并删除 Debian
wsl --unregister Debian

# 设置 Debian 为默认发行版
wsl --set-default Debian
wsl -s Debian

# 将 Debian 挪到喜欢的位置
wsl --mange Debian --move /path


# 将 Debian 设置为稀疏磁盘，允许自动回收磁盘空间
wsl --manage Debian -s true
wsl --manage Debian --set-sparse true


```

## 镜像导出

```shell

# 将 Debian 发行版导出为 debian-image.tar
wsl --export Debian debian-image.tar 

# 将 Debian 发行版导出为 debian-image.vhdx
wsl --export Debian debian-image.vhdx --vhd

```

## 镜像导入

```shell

# 导入 debian-image.tar 镜像为 Debian-Main
wsl --import Debian-Main /path/to/install /path/to/debian-image.tar

# 导入 debian-image.vhdx 镜像为 Debian-Main
wsl --import Debian-Main /path/to/install /path/to/debian-image.vhdx --vhd

```

# 常见问题

- 导出后的镜像登录默认为root而不是之前设置的普通用户？

  编辑 /etc/wsl.conf，添加配置

  ```shell
  [user]
  default=username
  ```