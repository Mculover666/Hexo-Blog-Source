---
title: 桌面mini网络时钟教程 | 04-OLED显示时钟和温湿度
keywords: RT-Thread
tags: 制作过程
categories: 制作过程
abbrlink: 3603126757
summary: 使用RT-Thread Studio打造
date: 2020-02-20 18:03:56
---

# 1. 项目进度
桌面Mini时钟项目用来演示如何使用RT-Thread Stduio开发项目，整个项目的架构如下：
![](https://img-blog.csdnimg.cn/2020020311120887.png#pic_center) 

在前三篇博文中简单的介绍了RT-Thread Studio一站式工具，基于STM32L431RCT6这个芯片创建工程，并修改时钟为使用外部时钟，以及添加SHT3x软件包获取温湿度传感器数据，最后添加了ESP8266设备连接网络，使用NTP服务器进行网络对时。

> - [使用RT-Thread Studio DIY 迷你桌面时钟（一）| 基于STM32芯片创建工程](https://blog.csdn.net/Mculover666/article/details/104146623)
>- [使用RT-Thread Studio DIY 迷你桌面时钟（二）| 获取温湿度传感器数据（I2C设备驱动+SHT3x软件包）](https://mculover666.blog.csdn.net/article/details/104153715)
>- [使用RT-Thread Studio DIY 迷你桌面时钟（三）| 获取NTP时间（at_device软件包 + netutils软件包）](https://mculover666.blog.csdn.net/article/details/104418075)

接下来添加u8g2软件包，驱动OLED显示数字时钟效果和温湿度效果。

# 2. 开启C++组件支持
使用U8G2软件包需要C++组件支持，在RT-Thread项目设置中开启C++组件，如图：
![](https://img-blog.csdnimg.cn/20200221091817570.png)
开启之后保存设置，软件会自动添加C++组件到工程中，编译没有问题：
![](https://img-blog.csdnimg.cn/20200221094621580.png)
# 3. 添加u8g2软件包并测试
c++组件测试编译没有问题之后，打开工程设置，搜索u8g2，添加软件包：
![](https://img-blog.csdnimg.cn/20200221094721981.png)
添加之后打开u8g2软件包配置，使能基本示例：
![](https://img-blog.csdnimg.cn/20200221094848273.png)
保存配置，软件会自动构建工程，打开示例文件，文件位置如图：
![](https://img-blog.csdnimg.cn/20200221095017406.png)
在示例文件最开始根据实际情况配置引脚编号：
![](https://img-blog.csdnimg.cn/20200221095216644.png)
配置之后，编译，下载，在串口控制台中运行该线程，如图：
![](https://img-blog.csdnimg.cn/20200221095426701.png)
线程运行起来之后OLED显示正常，测试完毕，如图：




# 4. 编写OLED显示线程
测试完毕后，在RT-Thread项目设置中关闭该示例文件，保存设置，等待软件自动构建工程完毕后，在application分组下创建一个用户文件`oled_display.cpp`文件，存放本项目中的OLED显示代码。

完整的代码如下：
```c
#include <rthw.h>
#include <rtthread.h>
#include <rtdevice.h>
#include <U8g2lib.h>
#include <stdio.h>

#include <drv_soft_i2c.h>

extern "C"
{
#include <sht3x.h>
}
extern "C"
{
sht3x_device_t sht3x_init(const char *i2c_bus_name, rt_uint8_t sht3x_addr);
rt_err_t sht3x_read_singleshot(sht3x_device_t dev);
}


#define OLED_I2C_PIN_SCL                    7   // PA7
#define OLED_I2C_PIN_SDA                    20  // PB4

static U8G2_SSD1306_128X64_NONAME_F_SW_I2C u8g2(U8G2_R0,\
                                         /* clock=*/ OLED_I2C_PIN_SCL,\
                                         /* data=*/ OLED_I2C_PIN_SDA,\
                                         /* reset=*/ U8X8_PIN_NONE);

#define SUN 0
#define SUN_CLOUD  1
#define CLOUD 2
#define RAIN 3
#define THUNDER 4

static void drawWeatherSymbol(u8g2_uint_t x, u8g2_uint_t y, uint8_t symbol)
{
  // fonts used:
  // u8g2_font_open_iconic_embedded_6x_t
  // u8g2_font_open_iconic_weather_6x_t
  // encoding values, see: https://github.com/olikraus/u8g2/wiki/fntgrpiconic

  switch(symbol)
  {
    case SUN:
      u8g2.setFont(u8g2_font_open_iconic_weather_6x_t);
      u8g2.drawGlyph(x, y, 69);
      break;
    case SUN_CLOUD:
      u8g2.setFont(u8g2_font_open_iconic_weather_6x_t);
      u8g2.drawGlyph(x, y, 65);
      break;
    case CLOUD:
      u8g2.setFont(u8g2_font_open_iconic_weather_6x_t);
      u8g2.drawGlyph(x, y, 64);
      break;
    case RAIN:
      u8g2.setFont(u8g2_font_open_iconic_weather_6x_t);
      u8g2.drawGlyph(x, y, 67);
      break;
    case THUNDER:
      u8g2.setFont(u8g2_font_open_iconic_embedded_6x_t);
      u8g2.drawGlyph(x, y, 67);
      break;
  }
}

static void drawWeather(uint8_t symbol, int degree)
{
  drawWeatherSymbol(0, 63, symbol);
  u8g2.setFont(u8g2_font_logisoso32_tf);
  u8g2.setCursor(55, 63);
  u8g2.print(degree);
  u8g2.print("C");
}
static void drawHumidity(uint8_t symbol, int humidity)
{
  drawWeatherSymbol(0, 63, symbol);
  u8g2.setFont(u8g2_font_logisoso32_tf);
  u8g2.setCursor(55, 63);
  u8g2.print(humidity);
  u8g2.print("%");
}


void oled_display()
{
    u8g2.begin();
    u8g2.clearBuffer();

    u8g2.setFont(u8g2_font_logisoso32_tf);
    u8g2.setCursor(48+3, 42);
    u8g2.print("Hi~");     // requires enableUTF8Print()

    u8g2.setFont(u8g2_font_6x13_tr);            // choose a suitable font
    u8g2.drawStr(30, 60, "By Mculover666");   // write something to the internal memory
    u8g2.sendBuffer();

    sht3x_device_t  sht3x_device;
    sht3x_device = sht3x_init("i2c1", 0x44);

    rt_thread_mdelay(2000);

    int status = 0;
    char mstr[3];
    char hstr[3];
    time_t now;
    struct tm *p;
    int min = 0, hour = 0;
    int temperature = 0,humidity = 0;

    while(1)
    {
        switch(status)
        {
            case 0:
                now = time(RT_NULL);
                p=gmtime((const time_t*) &now);
                hour = p->tm_hour;
                min = p->tm_min;
                sprintf(mstr, "%02d", min);
                sprintf(hstr, "%02d", hour);


                u8g2.firstPage();
                do {
                     u8g2.setFont(u8g2_font_logisoso42_tn);
                     u8g2.drawStr(0,63,hstr);
                     u8g2.drawStr(50,63,":");
                     u8g2.drawStr(67,63,mstr);
                   } while ( u8g2.nextPage() );


                rt_thread_mdelay(5000);
                status = 1;
                break;
           case 1:
               if(RT_EOK == sht3x_read_singleshot(sht3x_device))
               {
                   temperature = (int)sht3x_device->temperature;
               }
               else
               {
                   temperature = 0;
               }
               u8g2.clearBuffer();
               drawWeather(SUN, temperature);
               u8g2.sendBuffer();
               rt_thread_mdelay(5000);
               status = 2;
               break;
           case 2:
               if(RT_EOK == sht3x_read_singleshot(sht3x_device))
              {
                   humidity = (int)sht3x_device->humidity;
              }
              else
              {
                  humidity = 0;
              }
              u8g2.clearBuffer();
              drawHumidity(RAIN, humidity);
              u8g2.sendBuffer();
              rt_thread_mdelay(5000);
              status = 0;
              break;
        }
    }
}
MSH_CMD_EXPORT(oled_display, oled start);
```
最后的效果视频太大，放不上来，暂且用图片代替：

![](https://img-blog.csdnimg.cn/20200221142507452.png)

![](https://img-blog.csdnimg.cn/20200221142521218.png)
![](https://img-blog.csdnimg.cn/20200221142539777.png)
![](https://img-blog.csdnimg.cn/20200221142557580.png)

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)




