---
title: 使用Arduino+VsCode的方式开发Nodemcu
tags:
  - arduino
  - esp8266
  - nodemcu
  - vscode
id: '84'
categories:
  - - 嵌入式开发
date: 2020-02-28 17:40:36
---

**前言** 为什么用Vscode？很简单，Arduino IDE的编辑器不好用。废话不多说，来看怎么配置。 **0、Nodemcu连接电脑** 根据你Nodemcu的版本打上串口驱动，如果是后期新购买的Nodemcu那驱动有两个版本，一个是CH340一个是CP2102。安装完驱动程序后将Nodemcu连接电脑，在电脑的设备管理器中找到端口选项，如下图相似则安装成功。 ![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/11e3e749bddf136.png) **1、安装配置ArduinoIDE** 在[arduino官网](http://www.arduino.cc)下载对应系统版本的安装文件，下载安装arduino IDE。 ![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/9cb2adac62db237.png) 打开arduino IDE，左上角File选项，找到Preferences，在Preferences窗口的Settings框中找到选项Additional Boards Manager URLs，添加保存链接：http://arduino.esp8266.com/stable/package\_esp8266com\_index.json ![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/75385e5626cb974.png) 左上角Tools选项，找到Board，Boards Manager... ，在Boards Manager框中搜索esp8266，下载ESP8266 Community，由于我国网络环境这里下载非常缓慢，需要小工具加速，这里不再赘述。 ![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/f6a54e83be0f8a2.png) **2、安装配置VsCode** [VsCode官网](https://code.visualstudio.com/)下载对应系统版本的安装包安装VsCode，打开VsCode，在左边Extensions选项中搜索下载Arduino插件。 ![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/063850dde8de024.png) 在左上角选项File->Preferences->Settings进入设置页面，在User或者Workspace中找到Arduino configuration，我们这里在User中配置，点击Edit in settings.json ![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/e7eb7fdf1744d55.png) 在settings.json文件中添加以下内容，根据你的实际情况做改动。然后保存settings.json

```json
"arduino.path": "D:\\Program Files (x86)\\Arduino", //这里是你安装Arduino的文件路径
"C_Cpp.intelliSenseEngine": "Tag Parser",
"editor.tabSize": 4,
"editor.insertSpaces": true,
"files.autoGuessEncoding": true,
"arduino.logLevel": "info",
"editor.detectIndentation": true,
```

接下来我们在电脑桌面创建一个blink文件夹，在VsCode中File->Open Folder打开这个文件夹，创建一个blink.ino文件 ![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/d728560e6db68b7.png) 在VsCode底部看到COM6（你前边设备管理器中对应的串口号），在COM6旁边找到对应板型，点击搜索Nodemcu，找到1.0版本选择保存 ![](/assets/images/9fb83cc008f143d.png) ![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/8a03f6905cc7ee9.png) 在刚刚创建的blink.ino文件中编写代码如下

```c
void setup()
{
    pinMode(LED_BUILTIN,OUTPUT);
}

void loop()
{
    digitalWrite(LED_BUILTIN,HIGH);
    delay(1000);
    digitalWrite(LED_BUILTIN,LOW);
    delay(1000);
}
```

保存blink.ino，找到右上角上传图标，将文件编译烧写到Nodemcu，烧写完成后Nodemcu上LED闪烁。 ![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/7092344df660ad5.png) ![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/3274fb883e94c2a.png)