---
title: Windows下Clion+OpenCV开发环境搭建
tags:
  - opencv
  - 图像处理
id: '350'
categories:
  - - 折腾记录
date: 2020-10-22 12:11:18
---

### 准备

*   [Clion](https://www.jetbrains.com/clion/ "Clion")
*   [MinGW](https://sourceforge.net/projects/mingw-w64/files/ "MinGW")
*   [Cmake](https://cmake.org/download/ "Cmake")
*   [OpenCV](https://opencv.org/releases/ "OpenCV")

### 步骤

1.  安装Clion
2.  安装MinGW并将MinGW\\bin添加到环境变量
3.  安装CMake
4.  解压OpenCV源码
5.  打开CMake,在第一个框中打开源码目录，第二个框为编译目录，点击Configure ![](https://cdn.site.taoidle.com/wp-content/uploads/2020/10/wp_editor_md_645ab42b1af5be15646efb3290064a98.jpg)再次点击Configure红色消失 ![](https://cdn.site.taoidle.com/wp-content/uploads/2020/10/wp_editor_md_e4af1ce913054df34ee86ef4408fde7e.jpg) 点击Generate ![](https://cdn.site.taoidle.com/wp-content/uploads/2020/10/wp_editor_md_3c90537c4260f5537ea63ea0f55eafc6.jpg)
6.  打开CMD进入到MinGW-build目录下，执行mingw32-make
7.  上一步执行完成后执行mingw32-make install，生成install文件夹 ![](https://cdn.site.taoidle.com/wp-content/uploads/2020/10/wp_editor_md_a5da3358b4fddbd6fc4dc897bd325052.jpg)

### Clion中使用OpenCV

*   Clion设置 ![](https://cdn.site.taoidle.com/wp-content/uploads/2020/10/wp_editor_md_ed7877bd36d1481e37c61faba10c778b.jpg)
*   CMakeLists.txt文件配置

```
cmake_minimum_required(VERSION 3.17)
project(test)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
set(OpenCV_DIR "C:\\OpenCV")
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")
find_package(OpenCV)
include_directories(${OpenCV_INCLUDE_DIRS})
add_executable(test camera_calibration_correct.cpp)
set(OpenCV_LIBS opencv_core opencv_imgproc opencv_highgui opencv_imgcodecs)
target_link_libraries(test ${OpenCV_LIBS})
```