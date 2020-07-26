---
title: EasyFlash | 让Flash成为小型 KV 数据库
keywords: EasyFlash
tags: 嵌入式开源项目精选专栏
categories: OpenSource
abbrlink: 514838439
summary: 嵌入式开源项目精选专栏
date: 2020-04-25 08:00:56
---

# 嵌入式开源项目精选专栏
本专栏由Mculover666创建，主要内容为寻找嵌入式领域内的优质开源项目，一是帮助开发者使用开源项目实现更多的功能，二是通过这些开源项目，学习大佬的代码及背后的实现思想，提升自己的代码水平，和其它专栏相比，本专栏的优势在于：

**不会单纯的介绍分享项目，还会包含作者亲自实践的过程分享，甚至还会有对它背后的设计思想解读**。

目前本专栏包含的开源项目有：

- [cJSON | 一个轻量级C语言JSON解析器](https://blog.csdn.net/Mculover666/article/details/103796256)
- [paho | 支持10种语言编写mqtt客户端，总有一款适合你！](https://blog.csdn.net/Mculover666/article/details/103935428)
- [MultiButton | 一个小巧简单易用的事件驱动型按键驱动模块](https://blog.csdn.net/Mculover666/article/details/104992661)
- [letter-shell | 一个功能强大的嵌入式shell](https://blog.csdn.net/Mculover666/article/details/105141286)
- [EasyLogger | 一款轻量级且高性能的日志库](https://blog.csdn.net/Mculover666/article/details/105371993)
- [SFUD | 一款串行 Flash 通用驱动库 ](https://blog.csdn.net/Mculover666/article/details/105516371)

如果您自己编写或者发现的开源项目不错，欢迎留言或者私信投稿到本专栏，分享获得双倍的快乐！

# 1. EasyFlash
本期给大家带来的开源项目是 EasyFlash，**可以让 Flash 成为小型 KV 数据库（Key-Value）**，作者armink，目前收获 975 个 star，遵循 MIT 开源许可协议。

EasyFlash是一款开源的轻量级嵌入式Flash存储器库，非常适合智能家居、可穿戴、工控、医疗、物联网等需要**断电存储功能**的产品，资源占用极低，并且支持各种 MCU 片上存储器。

目前 EasyFlash 支持以下功能：

- ENV：快速保存产品参数，支持 写平衡（磨损平衡） 及掉电保护功能；
- IAP：在线升级；
- LOG：无需文件系统，日志可直接存储在Flash上；

>项目地址：[https://github.com/armink/EasyFlash](https://github.com/armink/EasyFlash)

# 2. 移植EasyFlash
## 2.1. 移植思路
在移植过程中主要参考两个资料：项目的readme文档和demo工程。

对于这些开源项目，其实移植起来也就两步：

- ① 添加源码到裸机工程中；
- ② 实现需要的接口即可（擦、写、读、打印）；

## 2.2. 准备裸机工程
本文中我使用的是小熊派IoT开发套件，主控芯片为STM32L431RCT6：
![](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2Jsb2cvMjAxOTA5MTkvekh4NXhNWVVtU1cxLnBuZw?x-oss-process=image/format,png#pic_center)
板载Flash型号为`W25Q64JV`，大小64Mbit，与STM32的QSPI接口相连：
![](https://img-blog.csdnimg.cn/20200416112021185.png)

移植之前需要准备一份裸机工程，我使用STM32CubeMX生成，需要初始化以下配置：

- 配置一个串口用于打印信息
- printf重定向
- 配置SPI Flash通信接口（SPI或QSPI）
- 移植SFUD开源库（方便操作Flash）

SFUD移植过程请参考上一期文章：

- [SFUD | 一款串行 Flash 通用驱动库 ](https://blog.csdn.net/Mculover666/article/details/105516371)

# 2.3. 添加EasyFlash到工程中
![](https://img-blog.csdnimg.cn/20200420180805827.png)
② 在keil中添加 SFUD 组件的源码文件：

- `src\easyflash.c`：（必选）包含EasyFlash初始化方法；
- `src\ef_utils.c`：（必选）EasyFlash常用小工具；
- `port\ef_port.c`：（必选）EasyFlash移植接口；
- `src\ef_env.c`：Env（常规模式）相关操作接口及实现源码；

![](https://img-blog.csdnimg.cn/20200420181230492.png)
其它两个IAP和LOG相关的源码暂且不用添加。

③ 将`easyflash/inc`头文件路径添加到keil中：

![](https://img-blog.csdnimg.cn/20200420181328372.png)

## 2.4. 实现EasyFlash移植接口
EasyFlash的移植接口都已经写好了，在`ef_port.c`文件中，只需要在函数体中添加代码即可。

① 默认环境变量集合

产品上需要的默认环境变量集中定义在这里，当 flash 第一次初始化时会将默认的环境变量写入，采用 `void *` 类型，所以支持任意类型：
```c
/* default environment variables set for user */
static const ef_env default_env_set[] = {
	{"wifi_ssid","FAST_88A6", 0},	//字符串大小设置为0，会自动检测
	{"wifi_passwd","12345678", 0},
};
```

② EasyFlash初始化接口

在该接口中会传递默认环境变量，初始化EasyFlash移植所需的资源（比如SFUD库的初始化）：
```c
EfErrCode ef_port_init(ef_env const **default_env, size_t *default_env_size) {
    EfErrCode result = EF_NO_ERR;

    *default_env = default_env_set;
    *default_env_size = sizeof(default_env_set) / sizeof(default_env_set[0]);
	
	//SFUD库初始化
	sfud_init();

    return result;
}
```

③ 读取Flash接口

使用SFUD开源库提供的API实现该接口：
```c
EfErrCode ef_port_read(uint32_t addr, uint32_t *buf, size_t size) {
    EfErrCode result = EF_NO_ERR;

	//获取SFUD Flash设备对象
    const sfud_flash *flash = sfud_get_device_table() + SFUD_W25Q64_DEVICE_INDEX;

	//使用SFUD开源库提供的API实现Flash读取
    sfud_read(flash, addr, size, (uint8_t *)buf);

    return result;
}
```

④ 擦除Flash接口

使用SFUD开源库提供的API实现该接口：
```c
EfErrCode ef_port_erase(uint32_t addr, size_t size) {
    EfErrCode result = EF_NO_ERR;
    sfud_err sfud_result = SFUD_SUCCESS;
	
	//获取SFUD Flash设备对象
    const sfud_flash *flash = sfud_get_device_table() + SFUD_W25Q64_DEVICE_INDEX;
    
    /* make sure the start address is a multiple of FLASH_ERASE_MIN_SIZE */
    EF_ASSERT(addr % EF_ERASE_MIN_SIZE == 0);
    
	//使用SFUD提供的API实现Flash擦除
    sfud_result = sfud_erase(flash, addr, size);

    if(sfud_result != SFUD_SUCCESS) {
        result = EF_ERASE_ERR;
    }

    return result;
}
```

⑤ 写入Flash接口

使用SFUD开源库提供的API实现该接口：
```c
EfErrCode ef_port_write(uint32_t addr, const uint32_t *buf, size_t size) {
    EfErrCode result = EF_NO_ERR;
    sfud_err sfud_result = SFUD_SUCCESS;
	
	//获取SFUD Flash设备对象
    const sfud_flash *flash = sfud_get_device_table() + SFUD_W25Q64_DEVICE_INDEX;

	//使用SFUD开源库提供的API实现
    sfud_result = sfud_write(flash, addr, size, (const uint8_t *)buf);

    if(sfud_result != SFUD_SUCCESS) {
        result = EF_WRITE_ERR;
    }

    return result;
}
```

⑥ 对环境变量缓冲区加锁/解锁

裸机时可以使用关闭/打开全局中断来上锁/解锁：
```c
/**
 * lock the ENV ram cache
 */
void ef_port_env_lock(void) {
    
	//关闭全局中断
   __disable_irq();
}

/**
 * unlock the ENV ram cache
 */
void ef_port_env_unlock(void) {
    
	//打开全局中断
	__enable_irq();
    
}
```

⑦ EasyFlash打印数据和日志接口

在该文件最顶部开辟一块打印数据缓冲区：
```c
//easyflash打印数据缓冲区
static char log_buf[128];
```

然后实现输出无固定格式的打印信息接口：
```c
void ef_print(const char *format, ...) {
    va_list args;

    /* args point to the first variable parameter */
    va_start(args, format);

    /* 实现数据输出 */
	vsprintf(log_buf, format, args);
    printf("%s", log_buf);
    
    va_end(args);
}
```
然后使用该函数去实现**调试信息日志打印接口**：
```c
void ef_log_debug(const char *file, const long line, const char *format, ...) {

#ifdef PRINT_DEBUG

    va_list args;

    /* args point to the first variable parameter */
    va_start(args, format);

    /* You can add your code under here. */
	  ef_print("[Flash](%s:%ld) ", file, line);
    /* must use vprintf to print */
    vsprintf(log_buf, format, args);
    ef_print("%s", log_buf);
    printf("\r");
    
    va_end(args);

#endif

}
```
最后实现普通日志信息打印接口：
```c
void ef_log_info(const char *format, ...) {
    va_list args;

    /* args point to the first variable parameter */
    va_start(args, format);

    /* You can add your code under here. */
	ef_print("[Flash]");
    /* must use vprintf to print */
    vsprintf(log_buf, format, args);
    ef_print("%s", log_buf);
    printf("\r");
    
    va_end(args);
}
```
## 2.5. 配置EasyFlash
EasyFlash的核心功能配置文件在`ef_cfg.h`，修改说明如下。

① 环境变量功能相关
![](https://img-blog.csdnimg.cn/20200421121813716.png)

② Flash擦除粒度和写入粒度
![](https://img-blog.csdnimg.cn/20200421122026329.png)
③ 备份区相关
![](https://img-blog.csdnimg.cn/20200421122409326.png)
④ 调试日志是否开启
![](https://img-blog.csdnimg.cn/2020042112250197.png)
至此，EasyFlash移植、配置完成，接下来就可以愉快的使用了！

# 3. 使用EasyFlash
使用时包含头文件：
```c
#include <easyflash.h>
```
## 3.1. 初始化EasyFlash
```c
EfErrCode easyflash_init(void);
```
该 API 会初始化的EasyFlash的各个组件，初始化后才可以使用别的API，第一次初始化的时候，会自动调用 ef_env_set_default 将定义的默认环境变量保存到Flash。

在main函数中初始化EasyFlash：
```c
/* USER CODE BEGIN 2 */
//初始化EasyFlash
ret = easyflash_init();
if(ret != EF_NO_ERR)
{
	printf("EasyFlash init fail, EfErrCode = %d.r\n", ret);
}

/* USER CODE END 2 */
```
## 3.2. 环境变量操作API
在 V4.0 以后，环境变量在 EasyFlash 底层都是按照二进制数据格式进行存储，即 blob 格式 ，这样上层支持传入任意类型。

① 获取 blob 类型环境变量
```c
size_t ef_get_env_blob(const char *key, void *value_buf, size_t buf_len, size_t *save_value_len);
```
其中参数的意义如下：

- key：环境变量名称
- value_buf：存放环境变量的缓冲区
- buf_len：该缓冲区的大小
- save_value_len：返回该环境变量实际存储在 flash 中的大小
- 返回值：成功存放至缓冲区中的数据长度

② 设置 blob 类型环境变量

使用该API可以对环境变量完成如下操作：

- 增加 ：当环境变量表中不存在该名称的环境变量时，则会执行新增操作；
- 修改 ：入参中的环境变量名称在当前环境变量表中存在，则把该环境变量值修改为入参中的值；
- 删除：当入参中的value为NULL时，则会删除入参名对应的环境变量。

```c
EfErrCode ef_set_env_blob(const char *key, const void *value_buf, size_t buf_len);
```
其中参数的意义如下：

- key：环境变量名称
- value_buf：环境变量值缓冲区
- buf_len：缓冲区长度，即值的长度

## 3.3. 测试读取默认环境变量
在main.c中编写一个EasyFlash测试函数：
```c
void test_env(void)
{
	char wifi_ssid[20] = {0};
	char wifi_passwd[20] = {0};
	size_t len = 0;
	
	/* 读取默认环境变量值 */
	//环境变量长度未知，先获取 Flash 上存储的实际长度 */
	ef_get_env_blob("wifi_ssid", NULL, 0, &len);
	//获取环境变量
	ef_get_env_blob("wifi_ssid", wifi_ssid, len, NULL);
	//打印获取的环境变量值
	printf("wifi_ssid env is:%s\r\n", wifi_ssid);
	
	//环境变量长度未知，先获取 Flash 上存储的实际长度 */
	ef_get_env_blob("wifi_passwd", NULL, 0, &len);
	//获取环境变量
	ef_get_env_blob("wifi_passwd", wifi_passwd, len, NULL);
	//打印获取的环境变量值
	printf("wifi_passwd env is:%s\r\n", wifi_passwd);
	
	/* 将环境变量值改变 */
	ef_set_env_blob("wifi_ssid", "SSID_TEST", 9);
	ef_set_env_blob("wifi_passwd", "66666666", 8);

}
```
在main函数的初始化代码之后**调用**该函数，编译下载之后，在串口终端中可以看到读取结果：
![](https://img-blog.csdnimg.cn/20200421145455429.png)
此时环境变量已经被修改，直接复位开发板，可以看到读取出的新值：
![](https://img-blog.csdnimg.cn/20200421145605217.png)
## 3.4. Easyflash和letter-shell的结合
EasyFlash在测试阶段需要不断的设置环境变量、读取环境变量、开发板重新上电，这个特点刚好可以应用letter-shell，直接将两个常用函数导出为命令，在串口命令行测试。

letter-shell的移植过程请参考第2期：

- [letter-shell | 一个功能强大的嵌入式shell](https://blog.csdn.net/Mculover666/article/details/105141286)

移植之后将读取环境变量的API封装，导出到命令列表中：
```c
/* USER CODE BEGIN 4 */
int getenv(const char *key)
{
	size_t len = 0;
	char buf[20] = {0};
	
	//获取长度
	ef_get_env_blob(key, NULL, 0, &len);
	
	if(len == 0)
	{
		//环境变量不存在
		printf("evn %s is not exist\r\n", key);
		
		return -1;
	}
	else if(len < 20)
	{
		//环境变量值超长
		printf("buf size is not enough, len is %d\r\n", len);
		
		return -1;
	
	}
	else
	{
		//获取环境变量值
		ef_get_env_blob(key, buf, len, NULL);
		printf("read env %s, value is:%s\r\n", key, buf);
		
		return 0;
	}
}

SHELL_EXPORT_CMD(SHELL_CMD_PERMISSION(0)|SHELL_CMD_TYPE(SHELL_TYPE_CMD_FUNC), getenv, getenv, getenv);
/* USER CODE END 4 */
```
编译下载之后，在串口终端中查看串口输出：
![](https://img-blog.csdnimg.cn/20200421151933889.png)

然后进行读取环境变量测试：
![](https://img-blog.csdnimg.cn/20200421152245850.png)
测试成功，同理，修改环境变量的API也可以进行封装，导出到命令列表中进行测试。

# 4. 设计思想解读
对于EasyFlash的全新版本设计，作者armink在仓库中写了一份文档，解铃还须系铃人，笔者的技术水平有限，直接放上作者的文档链接，欢迎感兴趣的读者阅读。

- [EasyFlash V4.0 ENV 功能设计与实现](https://mculover666.blog.csdn.net/article/details/105715982)

# 5. 项目工程源码获取和问题交流
目前我将EasyFlash源码、我移植到小熊派STM32L431RCT6开发板的工程源码上传到了QQ群里（包含好几份HAL库，QQ相对速度快点），**可以在QQ群里下载，有问题也可以在群里交流，当然也欢迎大家分享出来自己移植的工程到QQ群里**：
![](https://img-blog.csdnimg.cn/20200418104909667.png)
放上QQ群二维码：
![](https://img-blog.csdnimg.cn/20200321144543508.png)

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)
