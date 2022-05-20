---
title: manjaro npm换源
tags:
  - Linux
  - manjaro
  - nodejs
  - npm
id: '338'
categories:
  - - 折腾记录
date: 2020-07-02 21:29:20
---

## 安装

```shell
sudo pacman -S nodejs npm
```

## 换源

1.  方法一：
    
    ```shell
    npm config set registry https://registry.npm.taobao.org
    ```
    

* * *

2.  方法二： 安装cnpm代替npm
    
    ```shell
    sudo npm install -g cnpm --registry=https://registry.npm.taobao.org
    ```