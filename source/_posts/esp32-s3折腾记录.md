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

Esp32不需要引入其它东西，可以直接使用FreeRTOS

下面代码将摇杆模块示例作为一个任务，再添加一个RGB灯任务，都在ESP32的1核心上运行

```cpp
#include <Arduino.h>
#include <soc/soc.h>
#include <soc/rtc_cntl_reg.h>
#include "soc/timer_group_struct.h"
#include "soc/timer_group_reg.h"

#if CONFIG_FREERTOS_UNICORE
#define ARDUINO_RUNNING_CORE 0
#else
#define ARDUINO_RUNNING_CORE 1
#endif

#define RGB_R_PIN 37
#define RGB_G_PIN 36
#define RGB_B_PIN 35

#define BUTTON_K1 2

#define PS2_1_X_PIN 4
#define PS2_1_Y_PIN 5
#define PS2_1_Y_BUTTON 6

#define PS2_2_X_PIN 7
#define PS2_2_Y_PIN 15
#define PS2_2_Y_BUTTON 16

#define BEEP_PIN 9


void setup_led();
void setup_button();
void setup_ps2();
void setup_beep();

void beep(uint8_t PIN, uint8_t time, uint32_t time_len);
void turn_rgb(uint8_t R_PIN, uint8_t G_PIN, uint8_t B_PIN, uint32_t time_len);

void task_button_rgb(void *parameter);
void task_stick_beep(void *parameter);

int red_val = 255;
int green_val = 0;
int blue_val = 0;

TaskHandle_t task_1;
TaskHandle_t task_2;

void setup()
{
    Serial.begin(115200);

    setup_led();
    setup_button();
    setup_ps2();
    setup_beep();

    xTaskCreatePinnedToCore(task_stick_beep, "Task_1", CONFIG_ESP_MINIMAL_SHARED_STACK_SIZE, NULL, 1, &task_1, 1);
    xTaskCreatePinnedToCore(task_button_rgb, "Task_2", CONFIG_ESP_MINIMAL_SHARED_STACK_SIZE, NULL, 1, &task_2, 1);

}

void loop()
{
}

void task_button_rgb(void *parameter)
{
    (void) parameter;

    for (;;)
    {
        Serial.print("Task Button RGB running on core ");
        Serial.println(xPortGetCoreID());

        if (digitalRead(BUTTON_K1) == LOW)
        {
            turn_rgb(RGB_R_PIN, RGB_G_PIN, RGB_B_PIN, 5);
        }
    }
}

void task_stick_beep(void *parameter)
{
    (void) parameter;

    for (;;)
    {
        Serial.print("Task Stick Beep running on core ");
        Serial.println(xPortGetCoreID());

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
            beep(BEEP_PIN, 3, 50);
        }
    }
}

void setup_led()
{
    pinMode(RGB_R_PIN, OUTPUT);
    pinMode(RGB_G_PIN, OUTPUT);
    pinMode(RGB_B_PIN, OUTPUT);
}

void setup_button()
{
    pinMode(BUTTON_K1, INPUT_PULLUP);
    digitalWrite(BUTTON_K1, HIGH);
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
        Serial.print("for: ");
        Serial.println(i);
        digitalWrite(PIN, LOW);
        vTaskDelay(time_len);
        digitalWrite(PIN, HIGH);
        vTaskDelay(time_len);
    }
}

void turn_rgb(uint8_t R_PIN, uint8_t G_PIN, uint8_t B_PIN, uint32_t time_len)
{
    for (byte i = 0; i < 255; i++)
    {
        red_val--;
        green_val++;
        analogWrite(R_PIN, red_val);
        analogWrite(G_PIN, green_val);
        vTaskDelay(time_len);
    }
    for (byte i = 0; i < 255; i++)
    {
        green_val--;
        blue_val++;
        analogWrite(G_PIN, green_val);
        analogWrite(B_PIN, blue_val);
        vTaskDelay(time_len);
    }
    for (byte i = 0; i < 255; i++)
    {
        blue_val--;
        red_val++;
        analogWrite(B_PIN, blue_val);
        analogWrite(R_PIN, red_val);
        vTaskDelay(time_len);
    }
}

```

### TFT

在Arduino里，TFT LCD屏幕的驱动方式有好几种，这里选择的是TFT_eSPI这个库。
TFT_eSPI这个库使用起来非常方便，根据自己所使用的屏幕具体的参数在库中的User_Setup.h这个头文件里修改好就行了，具体的话需要修改屏幕的驱动、屏幕的分辨率、屏幕的所使用的引脚。

下面为头文件示例：

```cpp
#define USER_SETUP_INFO "User_Setup"


#define ST7735_DRIVER      // Define additional parameters below for this display
//#define ILI9163_DRIVER     // Define additional parameters below for this display
//#define S6D02A1_DRIVER
//#define RPI_ILI9486_DRIVER // 20MHz maximum SPI
//#define HX8357D_DRIVER
//#define ILI9481_DRIVER
//#define ILI9486_DRIVER
//#define ILI9488_DRIVER     // WARNING: Do not connect ILI9488 display SDO to MISO if other devices share the SPI bus (TFT SDO does NOT tristate when CS is high)
//#define ST7789_DRIVER      // Full configuration option, define additional parameters below for this display
//#define ST7789_2_DRIVER    // Minimal configuration option, define additional parameters below for this display
//#define R61581_DRIVER
//#define RM68140_DRIVER
//#define ST7796_DRIVER
//#define SSD1351_DRIVER
//#define SSD1963_480_DRIVER
//#define SSD1963_800_DRIVER
//#define SSD1963_800ALT_DRIVER
//#define ILI9225_DRIVER
//#define GC9A01_DRIVER

 #define TFT_WIDTH  128
 #define TFT_HEIGHT 160

 #define ST7735_REDTAB
// #define ST7735_BLACKTAB
// #define ST7735_REDTAB160x80   // For 160 x 80 display with 24 pixel offset



#define TFT_MOSI 35 // In some display driver board, it might be written as "SDA" and so on.
#define TFT_SCLK 36
#define TFT_CS   39  // Chip select control pin
#define TFT_DC   38  // Data Command control pin
#define TFT_RST  37  // Reset pin (could connect to Arduino RESET pin)
#define TFT_BL   40  // LED back-light


// Comment out the #defines below with // to stop that font being loaded
// The ESP8366 and ESP32 have plenty of memory so commenting out fonts is not
// normally necessary. If all fonts are loaded the extra FLASH space required is
// about 17Kbytes. To save FLASH space only enable the fonts you need!

#define LOAD_GLCD   // Font 1. Original Adafruit 8 pixel font needs ~1820 bytes in FLASH
#define LOAD_FONT2  // Font 2. Small 16 pixel high font, needs ~3534 bytes in FLASH, 96 characters
#define LOAD_FONT4  // Font 4. Medium 26 pixel high font, needs ~5848 bytes in FLASH, 96 characters
#define LOAD_FONT6  // Font 6. Large 48 pixel font, needs ~2666 bytes in FLASH, only characters 1234567890:-.apm
#define LOAD_FONT7  // Font 7. 7 segment 48 pixel font, needs ~2438 bytes in FLASH, only characters 1234567890:-.
#define LOAD_FONT8  // Font 8. Large 75 pixel font needs ~3256 bytes in FLASH, only characters 1234567890:-.
//#define LOAD_FONT8N // Font 8. Alternative to Font 8 above, slightly narrower, so 3 digits fit a 160 pixel TFT
#define LOAD_GFXFF  // FreeFonts. Include access to the 48 Adafruit_GFX free fonts FF1 to FF48 and custom fonts

// Comment out the #define below to stop the SPIFFS filing system and smooth font code being loaded
// this will save ~20kbytes of FLASH
#define SMOOTH_FONT


// #define SPI_FREQUENCY   1000000
// #define SPI_FREQUENCY   5000000
// #define SPI_FREQUENCY  10000000
// #define SPI_FREQUENCY  20000000
#define SPI_FREQUENCY  27000000
// #define SPI_FREQUENCY  40000000
// #define SPI_FREQUENCY  55000000 // STM32 SPI1 only (SPI2 maximum is 27MHz)
// #define SPI_FREQUENCY  80000000

// Optional reduced SPI frequency for reading TFT
#define SPI_READ_FREQUENCY  20000000

// The XPT2046 requires a lower SPI clock rate of 2.5MHz so we define that here:
#define SPI_TOUCH_FREQUENCY  2500000

```

### GUIslice

...