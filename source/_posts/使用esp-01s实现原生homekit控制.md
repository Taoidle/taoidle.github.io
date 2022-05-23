---
title: 使用ESP-01/ESP-01S实现原生Homekit控制
tags:
  - Apple
  - esp01
  - esp01s
  - esp8266
  - Homekit
  - 智能家居
  - 物联网
  - 苹果
id: '322'
categories:
  - - 嵌入式开发
date: 2020-06-03 23:32:55
---

#### 准备材料：

从左到右分别为ESP-01/ESP-01s芯片 、 继电器模块 、 ESP-01/ESP-01s烧录器 ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/1591193142-WechatIMG4-scaled-e1591193170554.jpeg)

#### 准备固件：

[haamain.bin](https://github.com/RavenSystem/haa/releases/download/2.4.4/haamain.bin "haamain.bin") 、 [rboot.bin](https://github.com/SuperHouse/esp-open-rtos/raw/master/bootloader/firmware_prebuilt/rboot.bin "rboot.bin") 、 [blank\_config.bin](https://github.com/SuperHouse/esp-open-rtos/raw/master/bootloader/firmware_prebuilt/blank_config.bin "blank_config.bin") 这里三个文件都需要下载 [项目地址](https://github.com/RavenSystem/esp-homekit-devices "项目地址")

#### 准备刷写软件：

[flash\_download\_tool](https://www.espressif.com/sites/default/files/tools/flash_download_tool_v3.8.5_0.zip "flash_download_tool")

#### 刷写固件：

![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_dc60f654062ffaf51a78c08acd2bba99.jpg) ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_17aaa84cfd343767b9ff2190f37112de.jpg) ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_ae5295ddb67f9d167fb0cbc6e273d42b.jpg) （这里COM口根据你实际选择） 点击START进行刷写 如下显示时为刷写完成 ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_ed0e07c1fd2fb9b069d8406cadf05f73.jpg) 找到HAA开头的WiFi并连接 ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_bc59846a058b421d0e9d6cb772b9d550.jpg) 在浏览器中输入 192.168.4.1进入配置页面 复制下面配置文件输入到红框中

```json
{"c":{"l":1,"b":[{"g":2,"t":5}]},"a":[{"t":1,"s":1,"0":{"r":[{"g":0,"v":1}]},"1":{"r":[{"g":0,"v":0}]},"b":[{"g":2}]}]}
```

![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_442a50992d699b66adc7d22091f84dd8.jpg) 选择你家的WiFi并在红框中输入密码 ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_3601abf44598877a7b93a94c13d64c0e.jpg) 点击Save保存，耐心等待7-10分钟后芯片开始正常工作

#### 配对苹果家庭

![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_6a43f5d933a297c680519a2aab5b25e3.jpg) ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/wp_editor_md_73a4d355793de9eb87f78bba7441b8c4.jpg) 配置文件意思

```json
{
  "c": {
    "l": 1,//设置状态LED的GPIO引脚
    "b": [//按钮配置
      {
        "g": 2,//按钮的GPIO引脚
        "t": 5//时长
      }
    ]
  },
  "a": [//附件配置
    {
      "t": 1,//物理类型
      "s": 1,//启动后初始状态
      "0": {//动作
        "r": [
          {
            "g": 0,//GPIO
            "v": 1//值
          }
        ]
      },
      "1": {//动作
        "r": [
          {
            "g": 0,//GPIO
            "v": 0//值
          }
        ]
      },
      "b": [
        {
          "g": 2
        }
      ]
    }
  ]
}

```