---
layout: 腾讯云函数随机图api实现
title: 腾讯云函数随机图api实现
date: 2022-07-10 19:11:32
categories:
  - - 折腾记录
tags: 
  - tencent cloud
  - scf
  - leancloud
  - upyun
  - cdn
  - javascript
  - nodejs
---

## 前言

为了提高博客在国内的访问速度，于是决定做一个部署在国内的随机图api。然后看了一下，又拍云可以白嫖免费的cdn和存储桶，leancloud白嫖数据存储，腾讯云函数备案资源包5年一百多四舍五入不要钱，最终约等于白嫖麻了，舒服～

注：虽然但是，leancloud数据存储api每天只能请求3万次，对于我这种小博客来说，问题不大。

## 正文

### 思路

简单来说，为了偷懒，和之前在cloudflare worker上的实现没区别，语言换成了javasript，框架换成express，数据存储换成leancloud。

至于为什么之前的项目用的是typescript而这个变成了javascript，主要原因leancloud的包在typescript和webpack一起用的时候会出莫名其妙的问题（其实是webpack出问题），赶时间不想排查原因。

#### 随机图接口

leancloud结构化数据中存储了一个class叫Api_Data，class中的object有api_key和api_value两个属性，其中有一个api_key属性值为KV_INDEX的object其属性api_value中存储了如下结构的数据，用于查找和索引：

```json
{
  "KV_INDEX": {
    "total_num": 300,
    "map_total": 3,
    "map_num": [
      "MAP_1_NUM",
      "MAP_2_NUM",
      "MAP_3_NUM"
    ],
    "map_list": [
      "MAP_KEY_1",
      "MAP_KEY_2",
      "MAP_KEY_3"
    ]
  }
}
```

存储了若干个表，表如下

```json
{
  "MAP_KEY_1": {
    "pic_1_hash": "pic_1_path",
    "pic_2_hash": "pic_2_path",
    "pic_3_hash": "pic_3_path",
    "pic_99_hash": "pic_99_path",
    "pic_100_hash": "pic_100_path"
  }
}
```

```json
{
  "MAP_KEY_2": {
    "pic_101_hash": "pic_101_path",
    "pic_102_hash": "pic_102_path",
    "pic_103_hash": "pic_103_path",
    "pic_199_hash": "pic_199_path",
    "pic_200_hash": "pic_200_path"
  }
}
```

```json
{
  "MAP_KEY_3": {
    "pic_201_hash": "pic_201_path",
    "pic_202_hash": "pic_202_path",
    "pic_203_hash": "pic_203_path",
    "pic_299_hash": "pic_299_path",
    "pic_300_hash": "pic_300_path"
  }
}
```

于是每次随机一张图会产生两次read操作，一次从索引中拿到map_key,一次从map中取数据

#### 数据更新接口

接受数据如下

```json
{
    "pic_1_hash": "pic_1_path",
    "pic_2_hash": "pic_2_path",
    "pic_3_hash": "pic_3_path",
    "pic_99_hash": "pic_99_path",
    "pic_100_hash": "pic_100_path"
  }
```

更新数据时步骤如下

1. 从kv中取索引，在从索引中获取到表中的key

2. 判断更新数据的key在表中时候存在，不存在则将该项添加到待更新表

3. 全部待更新数据添加到待更新表后，更新索引数据，最后把所有更新的数据写入到kv中


### 步骤

1. 在 [又拍云](https://www.upyun.com) 创建存储桶，绑定自定义域名，把资源上传到又拍云的存储桶中。在[又拍云联盟](https://www.upyun.com/league)加入开发者帮助计划，申请免费的云存储空间和CDN流量。

2. 在leancloud 国内版创建一个应用。
  
3. 把 [RandPiSCF](https://github.com/Taoidle/RandPicSCF) 部署到腾讯云函数。

    注意，需要配置一下环境变量(在leancloud你创建的应用->设置->应用凭证)：

    - LEANCLOUD_APP_ID leancloud中的AppID

    - LEANCLOUD_APP_KEY leancloud中的AppKey

    - LEANCLOUD_APP_MASTER_KEY leancloud中的MasterKey

    - LEANCLOUD_APP_SERVER_URL leancoud中REST API 服务器地址或者域名绑定中的api域名

    - PAGE_ITEM 数据存储中的分表

4. 随便写个脚本把图片信息丢上去完事 !!本来这里是有个rust写的程序的，用于将图片转为webp然后把hash值和路径丢到worker和scf上，但是由于目前比较忙，有空再写!!

## 结束语

其实就是前文 [使用cloudflare和jsdelivr做一个随机图api](https://www.taoidle.com/2022/05/24/使用cloudflare和jsdelivr做一个随机图api/) 的腾讯云函数偷懒实现版本，并没有对使用的资源做任何优化。后续有空优化一下，反正目前对我来说够用。
