---
layout: 使用cloudflare和jsdelivr做一个随机图api
title: 使用cloudflare和jsdelivr做一个随机图api
date: 2022-05-24 16:34:24
categories:
  - - 折腾记录
tags: 
  - cloudflare
  - cloudflare worker
  - github
  - cdn
  - typescript
  - nodejs
---
### 前言

博客最近进行迁移，原本使用wordpress，现迁移至hexo。从原来的腾讯云轻量应用服务器迁移到 cloudflare page。既然如此，那原本部署在腾讯云上使用python搭建的随即图api也没必要存在了。做随机图api的初衷只是因为想要自己喜欢的图。

注：博主的图片只有不到30张，小样本使用，数量较少。

### 正文

#### 思路

比较简单，cloudflare kv有1G空间，把hash和图片对应的路径存进kv里，然后worker从kv里随机取一个出来跳转到jsdelivr就行了

#### 步骤

1. 在github上创建一个空仓库用于存放图片，然后将自己喜欢的图片提交到该仓库即可，然后就可以在jsdelivr上取到该仓库的文件

    ```console
    https://cdn.jsdelivr.net/gh/github用户名/仓库名@分支
    ```
  
2. 克隆 [RandPicWorker](https://github.com/Taoidle/RandPicWorker) 这个项目，在项目根目录下执行yarn打好依赖。然后使用以下命令登录cloudflare

    ```console
    yarn run wrangler login
    ```

3. 在cloudflare上创建一个kv namespace

4. 填写项目里wrangler.toml文件缺少的信息

   ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/Snipaste_2022-05-25_22-30-06.png)

5. 执行package.json scripts里的publish命令

6. 随便写个脚本把图片信息丢上去完事 !!本来这里是有个rust写的程序的，用于将图片转为webp然后把hash值和路径丢到worker上，但是由于目前比较忙，有空再写!!

### 结束语

代码过于简单没啥好说的，好久没写博客了上来水一篇



------------------

## 更新

### 实现思路
#### 随机图接口

由于cloudflare kv的免费额度限制，write、delete、list 每天只有1000次，而read有10万次， 所以尽可能将kv操作使用read来实现。

kv中存储了一个index数据，用来查找数据，如下：

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

### 资源消耗


#### 随机图接口

于是每次随机一张图会产生两次read操作，一次从索引中拿到map_key,一次从map中取数据，以每天cloudflare kv 每天10万次免费的read，一天可以请求随机图接口5万次


#### 数据更新接口


假设kv中每个表存储1000条数据，kv中已有一个表存储100条数据，更新的数据有2000条，更新时会读取索引值一次，读取已存在的所有表（这里为一次），更新的信息为 已经存在的表1增加了900条数据，创建表2及表2 1000条数据，创建表3 及表3 100条数据，索引值更新。此时会对kv做4次写入，总共是2次读取，4次写入。
