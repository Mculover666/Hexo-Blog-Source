---
title: jsmn | 一个资源占用极小，解析速度最快的json解析器
keywords: jsmn
tags: 嵌入式开源项目精选专栏
categories: OpenSource
abbrlink: 1514366709
summary: 嵌入式开源项目精选专栏
date: 2020-05-25 08:00:56
---


![](https://img-blog.csdnimg.cn/20200321151234798.png)


# 1. jsmn
本期给大家带来的开源项目是 jsmn，**一个资源占用极小的json解析器，号称世界上最快**，作者zserge，目前收获 2.1K 个 star，遵循 MIT 开源许可协议。


jsmn主要有以下特性：

- 没有任何库依赖关系；
- 语法与C89兼容，代码可移植性高；
- 没有任何动态内存分配
- 极小的代码占用
- API只有两个，极其简洁

>项目地址：[https://github.com/zserge/jsmn](https://github.com/zserge/jsmn)

# 2. 移植jsmn
## 2.1. 移植思路
开源项目在移植过程中主要参考项目的readme文档，一般只需两步：

- ① 添加源码到裸机工程中；
- ② 实现需要的接口；

## 2.2. 准备裸机工程
本文中我使用的是小熊派IoT开发套件，主控芯片为STM32L431RCT6：
![](https://imgconvert.csdnimg.cn/aHR0cDovL21jdWxvdmVyNjY2LmNuL2Jsb2cvMjAxOTA5MTkvekh4NXhNWVVtU1cxLnBuZw?x-oss-process=image/format,png#pic_center)

移植之前需要准备一份裸机工程，我使用STM32CubeMX生成，需要初始化以下配置：

- 配置一个串口用于发送数据；
- printf重定向

具体过程可以参考：

- [STM32CubeMX_07 | 使用USART发送和接收数据（中断模式）](http://www.mculover666.cn/posts/1803605667/)
- [STM32CubeMX_09 | 重定向printf函数到串口输出的多种方法](http://www.mculover666.cn/posts/2251182441/)

# 2.3. 添加jsmn到工程中
① 复制jsmn源码到工程中：
![](https://img-blog.csdnimg.cn/20200519112146427.png)

② 将 jsmn.h 文件添加到keil中（**没有实质作用，方便编辑**）：
![](https://img-blog.csdnimg.cn/20200522141838534.png)
③ 添加jsmn头文件路径：
![](https://img-blog.csdnimg.cn/20200522141951848.png)

# 3. 使用jsmn解析json数据
## 3.1. 准备工作
① 包含jsmn头文件
使用时包含头文件，因为jsmn的函数定义也是在头文件中，所以第一次添加的时候，可以直接添加：
```c
/* USER CODE BEGIN Includes */
#include "jsmn.h"
#include <stdio.h>	//用于printf打印
#include <string.h> //用于字符串处理

/* USER CODE END Includes */
```
已经使用过之后，在别的文件中继续使用时，需要这样添加，且**顺序不可互换**：
```c
/* USER CODE BEGIN 0 */
#define JSMN_HEADER
#include "jsmn.h"	

/* USER CODE END 0 */
```
否则会造成函数重定义：
![](https://img-blog.csdnimg.cn/2020052309101761.png)

② 设置一段原始json数据
在main.c中设置原始的json数据，用于后续解析：
```c
/* USER CODE BEGIN PV */
static const char *JSON_STRING =
    "{\"user\": \"johndoe\", \"admin\": false, \"uid\": 1000,\n"
    "\"groups\": [\"users\", \"wheel\", \"audio\", \"video\"]}";
/* USER CODE END PV */
```
③ 开辟一块存放token的数组（token池）
jsmn中，每个数据段解析出来之后是一个token，关于token的详细解释，请参考下文第4.1小节。

```c
/* USER CODE BEGIN PV */

jsmntok_t t[128];

/* USER CODE END PV */
```
④ 编写在原始JSON数据中的字符串比较函数：
```c
static int jsoneq(const char *json, jsmntok_t *tok, const char *s) {
  if (tok->type == JSMN_STRING && (int)strlen(s) == tok->end - tok->start &&
      strncmp(json + tok->start, s, tok->end - tok->start) == 0) {
    return 0;
  }
  return -1;
}
```
## 3.2. 创建并初始化解析器
在main函数的开始创建解析器：
```c
/* USER CODE BEGIN 1 */
	int r;
	int i;
	
	jsmn_parser p;//jsmn解析器

/* USER CODE END 1 */
```
在随后外设初始化完成之后的代码中初始化解析器：
```c
/* USER CODE BEGIN 2 */
 
	jsmn_init(&p);

/* USER CODE END 2 */
```

## 3.3. 解析数据，获取token
```c
r = jsmn_parse(&p, JSON_STRING, strlen(JSON_STRING), t,sizeof(t) / sizeof(t[0]));

  if (r < 0) {
    printf("Failed to parse JSON: %d\n", r);
    return 1;
  }
  
  /* Assume the top-level element is an object */
  if (r < 1 || t[0].type != JSMN_OBJECT) {
    printf("Object expected\n");
    return 1;
  }
```
## 3.4. 逐个解析token
```c
/* Loop over all keys of the root object */
 for (i = 1; i < r; i++) 
 {
    if (jsoneq(JSON_STRING, &t[i], "user") == 0)
    {
      	/* We may use strndup() to fetch string value */
      	printf("- user: %.*s\n", t[i + 1].end - t[i + 1].start,
             JSON_STRING + t[i + 1].start);
      	i++;
    }
    else if (jsoneq(JSON_STRING, &t[i], "admin") == 0) 
    {
      	/* We may additionally check if the value is either "true" or "false" */
      	printf("- Admin: %.*s\n", t[i + 1].end - t[i + 1].start,
             JSON_STRING + t[i + 1].start);
      	i++;
    }
    else if (jsoneq(JSON_STRING, &t[i], "uid") == 0) 
    {
      	/* We may want to do strtol() here to get numeric value */
      	printf("- UID: %.*s\n", t[i + 1].end - t[i + 1].start,
             JSON_STRING + t[i + 1].start);
      	i++;
    }
    else if (jsoneq(JSON_STRING, &t[i], "groups") == 0) 
    {
      	int j;
      	printf("- Groups:\n");
      	if (t[i + 1].type != JSMN_ARRAY) 
      	{
        	continue; /* We expect groups to be an array of strings */
      	}
      	for (j = 0; j < t[i + 1].size; j++) 
      	{
        	jsmntok_t *g = &t[i + j + 2];
        	printf("  * %.*s\n", g->end - g->start, JSON_STRING + g->start);
      	}
     	 i += t[i + 1].size + 1;
    }
    else
    {
      	printf("Unexpected key: %.*s\n", t[i].end - t[i].start,
             JSON_STRING + t[i].start);
    }
  }
```
## 3.5. 解析结果
编译、下载到开发板，使用串口助手进行测试：
![](https://img-blog.csdnimg.cn/20200522155008406.png)
## 3.6. 内存对比
![](https://img-blog.csdnimg.cn/20200522154854214.png)

# 4. jsmn设计思想解读
## 4.1. jsmn对json数据项的抽象
jsmn对json数据中的每一个数据段都会抽象为一个结构体，称之为token，此结构体非常简洁：
```c
/**
 * JSON token description.
 * type		type (object, array, string etc.)
 * start	start position in JSON data string
 * end		end position in JSON data string
 */
typedef struct jsmntok {
  jsmntype_t type;
  int start;
  int end;
  int size;
#ifdef JSMN_PARENT_LINKS
  int parent;
#endif
} jsmntok_t;
```
在本实验中未开启`JSMN_PARENT_LINKS`，所以此结构体占用**16Byte大小**。

从结构体中的数据成员可以看出，**jsmn并不保存任何具体的数据内容**，仅仅记录：

- 数据项的类型
- 数据项数据段在原始json数据中的起始位置
- 数据项数据段在原始json数据中的结束位置

其中，数据项的类型支持4种：
```c
/**
 * JSON type identifier. Basic types are:
 * 	o Object
 * 	o Array
 * 	o String
 * 	o Other primitive: number, boolean (true/false) or null
 */
typedef enum {
  JSMN_UNDEFINED = 0,
  JSMN_OBJECT = 1,
  JSMN_ARRAY = 2,
  JSMN_STRING = 3,
  JSMN_PRIMITIVE = 4
} jsmntype_t;
```
## 4.2. jsmn如何解析出每个token
上述说到jsmn将每一个json数据段都抽象为一个token，那么jsmn是如何对整段json数据进行解析，得到每一个数据项的token呢？

jsmn解析器也是非常简洁的一个结构体：
```c
/**
 * JSON parser. Contains an array of token blocks available. Also stores
 * the string being parsed now and current position in that string.
 */
typedef struct jsmn_parser {
  unsigned int pos;     /* offset in the JSON string */
  unsigned int toknext; /* next token to allocate */
  int toksuper;         /* superior token node, e.g. parent object or array */
} jsmn_parser;
```

jsmn解析就是**将json数据逐个字符进行解析**，用pos数据成员来记录解析器当前的位置，当寻找到特殊字符时，就去之前我们定义的token数组（t）中申请一个空的token成员，将该token在数组中的位置记录在数据成员toknext中。

源码在下面的函数中，代码过多，暂且先不放：
```c
JSMN_API int jsmn_parse(jsmn_parser *parser, const char *js, const size_t len,
                        jsmntok_t *tokens, const unsigned int num_tokens);
```
下面用一个实例来看看token是怎么分配的。

缩短json原始数据：
```c
static const char *JSON_STRING =
    "{\"name\":\"mculover666\",\"admin\":false,\"uid\":1000}";
```
在解析之后将每个token打印出来：
```c
printf("[type][start][end][size]\n");
for(i = 0;i < r; i++)
{
	printf("[%4d][%5d][%3d][%4d]\n", t[i].type, t[i].start, t[i].end, t[i].size);
}
```
结果如下：
![](https://img-blog.csdnimg.cn/20200522154032155.png)
这段json数据解析出的token有7个：

① Object类型的token：`{\"name\":\"mculover666\",\"admin\":false,\"uid\":1000}`
② String类型的token：`"name"`、`"mculover666"`、`"admin"`、`"uid"`
③ Primitive类型的token：数字`1000`，布尔值`false`

![](https://img-blog.csdnimg.cn/20200522153959263.png)
## 4.3.  用户如何从token中提取值
在解析完毕获得这些token之后，需要根据token数量来判断是否解析成功：

① 返回的token数量<0：证明解析失败，返回值代表了错误类型：
```c
enum jsmnerr {
  /* Not enough tokens were provided */
  JSMN_ERROR_NOMEM = -1,
  /* Invalid character inside JSON string */
  JSMN_ERROR_INVAL = -2,
  /* The string is not a full JSON packet, more bytes expected */
  JSMN_ERROR_PART = -3
};
```
② 判断第0个token是否是JSMN_OBJECT类型，如果不是，则证明解析错误。

③ 如果token数量大于1，则从第1个token开始判断**字符串是否与给定的键值对的名称相等**，若相等，则提取下一个token的内容作为该键值对的值。

# 5. 项目工程源码获取和问题交流
目前我将jsmn源码、我移植到小熊派STM32L431RCT6开发板的工程源码上传到了QQ群里（包含好几份HAL库，QQ相对速度快点），**可以在QQ群里下载，有问题也可以在群里交流，当然也欢迎大家分享出来自己移植的工程到QQ群里**：
![](https://img-blog.csdnimg.cn/20200516103153959.png)

放上QQ群二维码：
![](https://img-blog.csdnimg.cn/20200321144543508.png)

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)
