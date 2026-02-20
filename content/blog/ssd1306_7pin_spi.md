---
title: "SSD1306 7 pin (SPI)"
date: 2024-04-25T11:33:29+05:30
draft: false
author: Jewel Mahanta
tags: [esp32, ssd1306, esp-idf, spi]
image: /ssd1306_7pin_spi.webp
description:
toc:
---
I am currently working on a new project and wanted to add a display. After searching for some decent (and cheap) displays I ordered myself a SSD1306 ([specifically this one](https://robu.in/product/0-96-oled-display-module-spii2c-128x64-7-pin-blue/)).

This specific display comes with SPI enabled by default (and we can set it up for I2C if we want). Since I had a pretty hard time trying setting it up with ESP-IDF, I figured I would share what I did.

## How to set it up?
My first attempt at interfacing this display was using [esp_lcd](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/peripherals/lcd.html) api provided by IDF. It has built in support for the SSD1306. However, even after hours and hours of wrecking my brain, reading through documentation, looking at examples, asking people on reddit and discord I was unable to get this to work in SPI. Another issue was that the vast majority of all examples floating around were in I2C. I realized that this is probably beyond me for the time being.

My next attempt at interfacing this display was using to simply switch to Arduino IDE and use the excellent [Adafruit_SSD1306](https://github.com/adafruit/Adafruit_SSD1306) library. If I still couldn’t get anything on the display it means that I need to replace it. Luckily, I was able to find some example code and finally got my display to work! Phew.

But how do I get this to work with IDF? After searching around for a while, I was able to find this library: [esp_idf-ssd1306](https://github.com/nopnop2002/esp-idf-ssd1306). The process to set this library for your own project is as follows:
```bash
# we clone this library to ~/esp-idf-ssd1306
$ cd ~
$ git clone https://github.com/nopnop2002/esp-idf-ssd1306.git
$ cd  /path/to/esp/project
# copy the components folder to the root of our project
$ cp -r ~/esp-idf-ssd1306/components ./
$ xed CMakeLists.txt # or any text editor
```
add this line to the opened file
```cmake
cmake_minimum_required(VERSION 3.16)

# <---- THIS LINE ----->
set(EXTRA_COMPONENT_DIRS ./components/ssd1306)
# <---- LINES BELOW ARE AUTO GENERATED SO DON'T TOUCH --->
include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(PROJECT_NAME)
```
Once you are done, you will have to open up the esp-idf terminal and do `idf.py menuconfig`. Scroll down to the `SSD1306 Configuration` option and select the Interface, Pins and Host:

![menuconfig 1](/menuconfig_1.png)

![menuconfig 2](/menuconfig_2.png)

![menuconfig 3](/menuconfig_3.png)

And you are set. Now you can just `#import ssd1306.h` and tinker around! Below is a minimal piece of code to get you started:
```c
#include "ssd1306.h"

void main(void) {
    SSD1306_t display_ssd1306;
    spi_master_init(&display_ssd1306, CONFIG_MOSI_GPIO, CONFIG_SCLK_GPIO,
                CONFIG_CS_GPIO, CONFIG_DC_GPIO, CONFIG_RESET_GPIO);
    ssd1306_init(&display_ssd1306, 128, 64);
    ssd1306_clear_screen(&display_ssd1306, false);
    ssd1306_contrast(&display_ssd1306, 0xff);

    ssd1306_display_text3(&display_ssd1306, 0, "Hello", 5, false);
    ssd1306_display_text(&display_ssd1306, 5, "World!", 6, false);
    while (1) {
        // do stuff
    }
}
```
Good luck!