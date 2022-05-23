---
title: CLion+PlatformIO环境搭建开发NodeMcu
tags:
  - arduino
  - CLion
  - esp8266
  - nodemcu
  - PlatformIO
id: '374'
categories:
  - - 嵌入式开发
comments: false
date: 2021-06-10 03:46:56
---

### 前言

JB家的东西实在是太好用了，丝毫不是因为我大半年没更新想水博客

### 准备环境

*   Python3
*   CLion

### 准备材料

*   NodeMcu
*   7 针0.96寸 OLED
*   杜邦线若干

### 正文

1.在Clion Plugins里搜索安装PlatformIO for CLion，安装完成后重启CLion

![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/1623260501-image-1623260500170.png)

2.安装PlatformIO Core (CLI)

*   Windows
    1.  下载 [get-platformio.py](https://raw.githubusercontent.com/platformio/platformio-core-installer/master/get-platformio.py "get-platformio.py")

2.  打开powershell
    
    ```shell
    cd C:\path\to\dir\where\is\located\script\get-platformio.py # get-platformio.py下载目录
    python.exe get-platformio.py # 运行脚本
    ```
    

*   Linux/MacOS
    
    1.  打开 terminal，下面命令三选一
    
    ```shell
    python3 -c "$(curl -fsSL https://raw.githubusercontent.com/platformio/platformio/master/scripts/get-platformio.py)"
    ```
    
    ```shell
    curl -fsSL https://raw.githubusercontent.com/platformio/platformio-core-installer/master/get-platformio.py -o get-platformio.py
    python3 get-platformio.py
    ```
    
    ```shell
    wget https://raw.githubusercontent.com/platformio/platformio-core-installer/master/get-platformio.py -O get-platformio.py
    python3 get-platformio.py
    ```
    

3.CLion创建PlatformIO NodeMcu项目

![file](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/1623263906-image-1623263905423.png)

![file](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/1623264001-image-1623263999430.png)

![file](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/1623264411-image-1623264410352.png)

到此项目创建完成

4.配置NodeMCU项目

*   修改CMakeLists.txt，将main.cpp添加到项目
    
    ![file](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/1623264624-image-1623264622615.png)
    
*   引入第三方库，如 U8g2
    
    在[platformio lib](https://platformio.org/lib "platformio lib")中搜索U8g2可以看到有两种安装方法
    
    1.  在CLion中打开terminal，输入 pio lib install "olikraus/U8g2"
    2.  编辑 platformio.ini 文件，在末尾中加入下面配置，编译时自动安装
    
    ```
    lib_deps = olikraus/U8g2 @ ^2.28.8
    ```
    
*   修改CMakeLists.txt，将U8g2库添加到项目中
    
    ![file](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/1623266913-image-1623266912048.png)
    
*   修改项目编译配置
    
    ![file](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/1623267020-image-1623267019679.png)
    
    把 build删除
    
    ![file](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/1623267165-image-1623267164884.png)
    
*   修改串口监视器
    
    ![file](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/1623267367-image-1623267366364.png)
    
    ![file](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/1623267453-image-1623267452000.png)
    

5.编写代码并编译上传到 NodeMCU

![file](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/1623267695-image-1623267693765.png)

![file](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/1623267853-image-1623267850897.png)