---
title: Raspberry Pi4b安装Docker
tags:
  - Docker
  - Raspberry Pi
  - Raspberry Pi4b
  - 树莓派
id: '404'
categories:
  - - 嵌入式开发
comments: false
date: 2022-04-09 15:22:46
---

1.  更新软件包
    
    ```bash
    sudo apt update
    sudo apt upgrade
    ```
    
2.  通过运行以下命令下载并运行官方 Docker 设置脚本
    
    ```bash
    curl -sSL https://get.docker.com  sh
    ```
    
3.  将pi用户添加到docker组中
    
    ```bash
    sudo usermod -aG docker pi
    ```