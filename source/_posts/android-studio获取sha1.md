---
title: Android Studio获取SHA1
tags:
  - android
  - android studio
id: '311'
categories:
  - - 折腾记录
date: 2020-05-10 16:11:21
---

## 前言

使用高德地图，腾讯地图等地图api都需要用到SHA1值

## 步骤

1.  检查Android Studio工程下时候存在keystore类型文件，如果不存在则执行命令

```shell
keytool -v -list -keystore keystore文件路径
```

会出现错误

```shell
keytool 错误: java.lang.Exception: 密钥库文件不存在
```

2.  创建密钥库文件

在Android Studio中打开Terminal，这里以创建一个debug.keystore为例，输入命令

```shell
 keytool -genkey -v -keystore debug.keystore -alias androiddebugkey -keyalg RSA -validity 10000
```

填写注册信息（可随意填写），设置密码，密码需记住后续获取SHA1需要用到密码 ![](https://cdn.site.taoidle.com/wp-content/uploads/2020/05/wp_editor_md_dda6a1e249ae12028b5cb20d65b16d4c.jpg) 查看debug.keystore是否生成，得到SHA1值（红框部分）

```shell
keytool -list -v -keystore debug.keystore
```

![](https://cdn.site.taoidle.com/wp-content/uploads/2020/05/wp_editor_md_523dfce26e74a850d2cd1eda3171290a.jpg)