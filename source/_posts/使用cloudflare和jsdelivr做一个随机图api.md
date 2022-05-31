---
layout: 使用cloudflare和jsdelivr做一个随机图api
title: jsdelivr做一个随机图api
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