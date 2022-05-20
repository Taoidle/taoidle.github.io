---
title: Python图像处理 —— 空域信息隐藏（LSB替换）
tags:
  - opencv
  - python
  - 原创教程
  - 图像处理
  - 教程
id: '336'
categories:
  - - 计算机视觉
date: 2020-07-02 14:40:04
---

## 算法描述

*   LSB即最低有效位，全称为Leastsignificant Bit，LSB隐写技术即就是通过修改RGB颜色分量的最低位二进制，进行信息的储存。

## 算法实现

*   LSB规律须知：奇数二进制最低有效位为1，偶数为0
*   嵌入过程：先将嵌入信息转换为二进制流，逐个像素进行替换
*   提取过程：先对像素进行提取，再将二进制流转为文本

## 算法缺点

1.  不可逆
2.  脆弱、不鲁棒

## Python代码

```python
import numpy as np
import cv2


def encode(s, width=8):
    bin_str = ''.join([(bin(c).replace('0b', '')).zfill(width) for c in s.encode(encoding="utf-8")])
    return bin_str


def decode(s):
    s = [np.uint8(c) for c in s]
    bin_str = ''.join([bin(b & 1).strip('0b').zfill(1) for b in s])
    str = np.zeros(np.int(len(s) / 8)).astype(np.int)
    for i in range(0, len(str)):
        str[i] = int('0b' + bin_str[(8 * i):(8 * (i + 1))], 2)
    return bytes(str.astype(np.int8)).decode()


def lsb_embed(img, s):
    if len(img.shape) == 3:
        # 获取通道数F
        width, height, channel = img.shape
        s = encode(s)
        for i in range(len(s)):
            x = i // width
            y = i % width
            channel = i // (height * width)
            value = img[x, y, channel]
            if (value % 2) == int(s[i]):
                pass
            if (value % 2) > int(s[i]):
                img[x, y, channel] = value - 1
            if (value % 2) < int(s[i]):
                img[x, y, channel] = value + 1
    else:
        width, height = img.shape
        s = encode(s)
        for i in range(len(s)):
            x = i // width
            y = i % width
            value = img[x, y]
            if (value % 2) == int(s[i]):
                pass
            if (value % 2) > int(s[i]):
                img[x, y] = value - 1
            if (value % 2) < int(s[i]):
                img[x, y] = value + 1
    return img, len(s)


def lsb_extract(img, num):
    if len(img.shape) == 3:
        # 获取通道数F
        width, height, channel = img.shape
        s = ''
        for i in range(num):
            x = i // width
            y = i % width
            channel = i // (height * width)
            value = img[x, y, channel]
            if value % 2 == 0:
                s += '0'
            else:
                s += '1'
    else:
        width, height = img.shape
        s = ''
        for i in range(num):
            x = i // width
            y = i % width
            value = img[x, y]
            if value % 2 == 0:
                s += '0'
            else:
                s += '1'
    return decode(s)


if __name__ == '__main__':
    # 嵌入
    msg = "hello world"
    pic_origin = cv2.imread('lenna.bmp')
    pic_embed, msg_len = lsb_embed(pic_origin, msg)
    cv2.imwrite('pic_embed.bmp', pic_embed)
    # 提取
    pic_embed = cv2.imread('pic_embed.bmp')
    msg_extract = lsb_extract(pic_embed, msg_len)
    print(msg_extract)

```

## 算法改进

1.  提取时无需输入信息长度 思路：将长度信息保存在图片某位置，在提取时先提取长度信息，再提取嵌入信息
2.  随机位置嵌入 思路：生成随机数即可，将随机数种子保存，用于保存信息的位置范围排除，其它位置范围里生成随机数