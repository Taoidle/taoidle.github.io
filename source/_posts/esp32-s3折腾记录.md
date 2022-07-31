---
layout: ESP32-S3 折腾记录
title: ESP32-S3 折腾记录
date: 2022-07-30 12:00:24
categories:
  - - 折腾记录
  - - 嵌入式开发
tags: 
  - esp32
  - esp32-s3-devkitc
  - platformio
  - arduino
---

## 前言

开发板：ESP32-S3-DEVKIT-C

开发环境：Clion + Platformio

开发方式：Arduino

## 正文

### 关于 ESP32-S3

为什么选择使用ESP32-S3？

在TMD 芯片贵上天的今天，一片STM32F103C8T6也要接近20块钱，在加一个NF24L01价格上跟一片ESP32-S3-WROOM-1/ESP32-S3-WROOM-1U N16R8差不多，但是资源上被ESP32-S3甩了10086条街。

先看一下ESP32-S3-WROOM-1 N16R8这个模组(ESP32-S3-WROOM-1 采用 PCB 板载天线，ESP32-S3-WROOM-1U 采用连接器连接外部天线，资源一致)

- Xtensa® 32 位 LX7 双核处理器，五级流水线架构，主频高达 240 MHz
- 2.4 GHz WiFi & Bluetooth 5
- PSRAM：8MB
- FALSH: 16MB
- GPIO: 45
- SPI: 4
- I2C: 2
- I2S: 2
- ADC: 12
- TOUCH: 14

除此之外，ESP32-S3安全机制也非常完善

- 安全启动
- Flash 加密
- 4-Kbit OTP，用户可用的高达 1792 位 
- 加密硬件加速器：
    - AES-128/256 (FIPS PUB 197)
    - Hash (FIPS PUB 180-4)
    - RSA
    - 随机数生成器 (RNG)
    - HMAC
    - 数字签名

如此丰富的硬件资源足够做很多“大玩具”，是折腾用的不二选择，以下内容记录ESP32-S3里踩的坑，不定时更新...

下面为 ESP32-S3-DEVKIT-C 开发板引脚定义图

![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/ESP32-S3_DevKitC-1_pinlayout_v1.1.jpg)



### SPI SD卡

官方示例如下，在官方示例中引脚为默认，在实际场景中往往需要自定义引脚。

```cpp
/*
 * Connect the SD card to the following pins:
 *
 * SD Card | ESP32
 *    D2       -
 *    D3       SS
 *    CMD      MOSI
 *    VSS      GND
 *    VDD      3.3V
 *    CLK      SCK
 *    VSS      GND
 *    D0       MISO
 *    D1       -
 */
#include "FS.h"
#include "SD.h"
#include "SPI.h"

void listDir(fs::FS &fs, const char * dirname, uint8_t levels){
    Serial.printf("Listing directory: %s\n", dirname);

    File root = fs.open(dirname);
    if(!root){
        Serial.println("Failed to open directory");
        return;
    }
    if(!root.isDirectory()){
        Serial.println("Not a directory");
        return;
    }

    File file = root.openNextFile();
    while(file){
        if(file.isDirectory()){
            Serial.print("  DIR : ");
            Serial.println(file.name());
            if(levels){
                listDir(fs, file.path(), levels -1);
            }
        } else {
            Serial.print("  FILE: ");
            Serial.print(file.name());
            Serial.print("  SIZE: ");
            Serial.println(file.size());
        }
        file = root.openNextFile();
    }
}

void createDir(fs::FS &fs, const char * path){
    Serial.printf("Creating Dir: %s\n", path);
    if(fs.mkdir(path)){
        Serial.println("Dir created");
    } else {
        Serial.println("mkdir failed");
    }
}

void removeDir(fs::FS &fs, const char * path){
    Serial.printf("Removing Dir: %s\n", path);
    if(fs.rmdir(path)){
        Serial.println("Dir removed");
    } else {
        Serial.println("rmdir failed");
    }
}

void readFile(fs::FS &fs, const char * path){
    Serial.printf("Reading file: %s\n", path);

    File file = fs.open(path);
    if(!file){
        Serial.println("Failed to open file for reading");
        return;
    }

    Serial.print("Read from file: ");
    while(file.available()){
        Serial.write(file.read());
    }
    file.close();
}

void writeFile(fs::FS &fs, const char * path, const char * message){
    Serial.printf("Writing file: %s\n", path);

    File file = fs.open(path, FILE_WRITE);
    if(!file){
        Serial.println("Failed to open file for writing");
        return;
    }
    if(file.print(message)){
        Serial.println("File written");
    } else {
        Serial.println("Write failed");
    }
    file.close();
}

void appendFile(fs::FS &fs, const char * path, const char * message){
    Serial.printf("Appending to file: %s\n", path);

    File file = fs.open(path, FILE_APPEND);
    if(!file){
        Serial.println("Failed to open file for appending");
        return;
    }
    if(file.print(message)){
        Serial.println("Message appended");
    } else {
        Serial.println("Append failed");
    }
    file.close();
}

void renameFile(fs::FS &fs, const char * path1, const char * path2){
    Serial.printf("Renaming file %s to %s\n", path1, path2);
    if (fs.rename(path1, path2)) {
        Serial.println("File renamed");
    } else {
        Serial.println("Rename failed");
    }
}

void deleteFile(fs::FS &fs, const char * path){
    Serial.printf("Deleting file: %s\n", path);
    if(fs.remove(path)){
        Serial.println("File deleted");
    } else {
        Serial.println("Delete failed");
    }
}

void testFileIO(fs::FS &fs, const char * path){
    File file = fs.open(path);
    static uint8_t buf[512];
    size_t len = 0;
    uint32_t start = millis();
    uint32_t end = start;
    if(file){
        len = file.size();
        size_t flen = len;
        start = millis();
        while(len){
            size_t toRead = len;
            if(toRead > 512){
                toRead = 512;
            }
            file.read(buf, toRead);
            len -= toRead;
        }
        end = millis() - start;
        Serial.printf("%u bytes read for %u ms\n", flen, end);
        file.close();
    } else {
        Serial.println("Failed to open file for reading");
    }


    file = fs.open(path, FILE_WRITE);
    if(!file){
        Serial.println("Failed to open file for writing");
        return;
    }

    size_t i;
    start = millis();
    for(i=0; i<2048; i++){
        file.write(buf, 512);
    }
    end = millis() - start;
    Serial.printf("%u bytes written for %u ms\n", 2048 * 512, end);
    file.close();
}

void setup(){
    Serial.begin(115200);
    if(!SD.begin()){
        Serial.println("Card Mount Failed");
        return;
    }
    uint8_t cardType = SD.cardType();

    if(cardType == CARD_NONE){
        Serial.println("No SD card attached");
        return;
    }

    Serial.print("SD Card Type: ");
    if(cardType == CARD_MMC){
        Serial.println("MMC");
    } else if(cardType == CARD_SD){
        Serial.println("SDSC");
    } else if(cardType == CARD_SDHC){
        Serial.println("SDHC");
    } else {
        Serial.println("UNKNOWN");
    }

    uint64_t cardSize = SD.cardSize() / (1024 * 1024);
    Serial.printf("SD Card Size: %lluMB\n", cardSize);

    listDir(SD, "/", 0);
    createDir(SD, "/mydir");
    listDir(SD, "/", 0);
    removeDir(SD, "/mydir");
    listDir(SD, "/", 2);
    writeFile(SD, "/hello.txt", "Hello ");
    appendFile(SD, "/hello.txt", "World!\n");
    readFile(SD, "/hello.txt");
    deleteFile(SD, "/foo.txt");
    renameFile(SD, "/hello.txt", "/foo.txt");
    readFile(SD, "/foo.txt");
    testFileIO(SD, "/test.txt");
    Serial.printf("Total space: %lluMB\n", SD.totalBytes() / (1024 * 1024));
    Serial.printf("Used space: %lluMB\n", SD.usedBytes() / (1024 * 1024));
}

void loop(){

}
```

需要注意的是，这里SPI SD卡必须接5V，官方示例中接入为3.3V，在实际中不行，会报错。


`
sdcard_mount(): f_mount failed: (3) The physical drive cannot work
`

正确做法应该将SPI SD接入5V后，自己初始化SPI引脚后传入SD卡初始化

```cpp
#include <SPI.h>
#include <FS.h>
#include <SD.h>

// VCC 5V

#define SD_MISO     13
#define SD_MOSI     11
#define SD_SCLK     12
#define SD_CS       10


void setup() {
    Serial.begin(115200);

    SPIClass spi = SPIClass(HSPI);
    spi.begin(SD_SCLK, SD_MISO, SD_MOSI, SD_CS);


    if (!SD.begin(SD_CS, spi)) {
        Serial.println("Card Mount Failed");
        return;
    }
    uint8_t cardType = SD.cardType();

    if (cardType == CARD_NONE) {
        Serial.println("No SD card attached");
        return;
    }

    Serial.print("SD Card Type: ");
    if (cardType == CARD_MMC) {
        Serial.println("MMC");
    } else if (cardType == CARD_SD) {
        Serial.println("SDSC");
    } else if (cardType == CARD_SDHC) {
        Serial.println("SDHC");
    } else {
        Serial.println("UNKNOWN");
    }

    uint64_t cardSize = SD.cardSize() / (1024 * 1024);
    Serial.printf("SD Card Size: %lluMB\n", cardSize);

    Serial.println("initialisation done.");
}

void loop() {

}

```

### 遥杆模块

遥杆需要注意将摇杆模块上的轻触开关引脚上拉

以下代码实现读取两个遥杆的数据，当轻触开关被按下是触发蜂鸣器（有源低电平蜂鸣器）

```cpp
#include <Arduino.h>

#define PS2_1_X_PIN 4
#define PS2_1_Y_PIN 5
#define PS2_1_Y_BUTTON 6

#define PS2_2_X_PIN 7
#define PS2_2_Y_PIN 15
#define PS2_2_Y_BUTTON 16

#define BEEP_PIN 9

void setup_ps2();
void setup_beep();

void beep(uint8_t PIN, uint8_t time, uint32_t time_len);

void setup()
{
    Serial.begin(115200);
    Serial.println("setup ...");

    setup_ps2();
    setup_beep();

    Serial.println("setup ok");
}

void loop()
{
    uint16_t ps2_lx = analogRead(PS2_1_X_PIN);
    uint16_t ps2_ly = analogRead(PS2_1_Y_PIN);
    uint16_t ps2_lb = digitalRead(PS2_1_Y_BUTTON);
    uint16_t ps2_rx = analogRead(PS2_2_X_PIN);
    uint16_t ps2_ry = analogRead(PS2_2_Y_PIN);
    uint16_t ps2_rb = digitalRead(PS2_2_Y_BUTTON);

    Serial.print("lX: ");
    Serial.print(ps2_lx);
    Serial.print(" ");
    Serial.print("lY: ");
    Serial.print(ps2_ly);
    Serial.print(" ");
    Serial.print("lZ: ");
    Serial.println(ps2_lb);
    Serial.print("rX: ");
    Serial.print(ps2_rx);
    Serial.print(" ");
    Serial.print("rY: ");
    Serial.print(ps2_ry);
    Serial.print(" ");
    Serial.print("rZ: ");
    Serial.println(ps2_rb);

    if (ps2_lb == LOW || ps2_rb == LOW)
    {
        Serial.println("in if");
        beep(BEEP_PIN, 3, 50);
    }

}

void setup_ps2()
{
    // set ps2-1
    pinMode(PS2_1_X_PIN, INPUT); // ps2-1-x
    pinMode(PS2_1_Y_PIN, INPUT); // ps2-1-y
    pinMode(PS2_1_Y_BUTTON, INPUT_PULLUP); // ps2-1-button
    digitalWrite(PS2_1_Y_BUTTON, HIGH);

    // set ps2-2
    pinMode(PS2_2_X_PIN, INPUT); // ps2-2-x
    pinMode(PS2_2_Y_PIN, INPUT); // ps2-2-y
    pinMode(PS2_2_Y_BUTTON, INPUT_PULLUP); // ps2-2-button
    digitalWrite(PS2_2_Y_BUTTON, HIGH);
}

void setup_beep()
{
    // set bee
    pinMode(BEEP_PIN, OUTPUT);
    digitalWrite(BEEP_PIN, HIGH);
}

void beep(uint8_t PIN, uint8_t time, uint32_t time_len)
{
    Serial.print("beep: ");
    Serial.println(PIN);

    for (int i = 0; i < time; i++)
    {
        digitalWrite(PIN, LOW);
        delay(time_len);
        digitalWrite(PIN, HIGH);
        delay(time_len);
    }

}
```

### FreeRTOS



### GUIslice

...