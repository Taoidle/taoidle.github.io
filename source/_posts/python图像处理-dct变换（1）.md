---
title: Python图像处理 —— DCT变换
tags:
  - opencv
  - pyqt5
  - python
  - 图像处理
  - 教程
id: '309'
categories:
  - - 计算机视觉
date: 2020-05-03 00:06:38
---

# 图像的DCT变换

## python实现

这里去除了图像的低频部分，进行dct变换后得到图像的骨架

```python
import cv2
import numpy as np

# 读取图像
img = cv2.imread('../res/img/baboon.bmp')
# 灰度化
img = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
# 显示原图
cv2.imshow('img', img)
# 获取图像宽高
height, width = img.shape
# 对图像进行dct变换
img_dct = cv2.dct(np.array(img, np.float32))
# 去除部分低频信息
img_dct[0:100, 0:100] = 0
# 进行反dct变换
img_idct = np.array(cv2.idct(img_dct), np.uint8)
# 显示图像
cv2.imshow('img_dct', np.array(img_dct, np.uint8).reshape(width, height))
cv2.imshow('img_idct', img_idct.reshape(width, height))
cv2.waitKey(0)

```

![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_f47ba882139a0929c17e72e8d408a598.jpg) ![]https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master(/assets/images/wp_editor_md_4d2787e275882434e47a8ac40c2d711a.jpg) ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_927e11e4879955a3bf4fdebb01ff5b73.jpg)

## CV Tool小工具中的操作

### 灰度图DCT

读取图像 ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_79f735a75273ec5494159dfe4170b128.jpg) 进行灰度化 ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_12d55d2288111f59e2fe8f7937987e5c.jpg) 选定DCT去除的信息，进行DCT变换 ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_f26670cf8f60a325901e9e8217616207.jpg) 得到效果图 ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_3b7437fbd0e276fcc1b281a85fd0a5da.jpg)

### 彩色图像DCT变换

打开图像 ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_cd0232d0bcdb5fa3abe9279e639b63ee.jpg) 选定DCT去除的信息，进行DCT变换 ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_392cc8c9f36cf5dadbb8fe641951f85e.jpg) 得到效果图 ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_44c76d239bee25c7f0dfcaa3e9352451.jpg)