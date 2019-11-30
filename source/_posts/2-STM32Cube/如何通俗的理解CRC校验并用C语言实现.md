---
title: 如何通俗的理解CRC校验并用C语言实现
tags: CRC校验
categories: 算法实现
abbrlink: 1935373145
date: 2019-08-06 10:16:24
---

本篇文章通俗的讲解了何谓CRC？如何生成CRC码以及如何使用CRC码校验，最后采用**多种思路**用C语言实现CRC校验。

![mark](http://mculover666.cn/image/20190829/a5uVezmqAgdw.png?imageslim)

<!--more-->

# 背景
最近在研究温湿度传感器SHT30，该传感器使用IIC接口进行通信，在读取数据时，该传感器**分别**在发送 `16bit` 的温度和湿度数据后，发送了 `8bit` 的CRC校验数据，如图所示：

![mark](http://mculover666.cn/image/20190808/OiN41VkYPDat.png?imageslim)


那么，问题来了：

- **CRC是个什么东东？**
- **如何计算得到CRC码？**
- **如何使用CRC码校验？**
- **如何用C语言实现该CRC校验？**

# CRC到底是个什么东东

CRC学名叫做**循环冗余校验**，全称 `cyclic redundancy check`，这个词有两个含义：

- 循环冗余校验**功能**：对要传送的数据进行多项式计算，并将所得结果跟着传送数据后发送，接收端再次进行校验；
- 循环冗余校验**码**：对要传送的数据进行多项式计算后得到的值称为循环冗余校验码。

等等，好像跑题了，这个是官方解释，那么，我们该如何通俗的理解呢？接下里我带领大家在实战中领悟CRC的奥秘~

# 计算CRC码之前的准备工作

## 关键点1 —— 模2除法
在CRC校验规则中，原始数据与给定的除数之间进行**模2除法**得到CRC码，发送端将原始数据和CRC码一起传送到接收端，接收端再

那么，模2除法的规则是怎样的呢？它和普通除法有什么不一样的呢？

模2除法与普通的算术除法类似，但是它有两个区别：

- 不向上借位；
- 不比较除数和被除数的相同位数值的大小，只要以相同位数进行相除即可；

所以在模2除法中二进制运算结果如下：

- 1 - 1 = 0
- 1 - 0 = 1
- 0 - 1 = 1
- 0 - 0 = 1

乍一看，这个运算规则是不是似曾相识~这就是我们熟悉的**异或运算**，可以总结出“模2除法”的规则：

    模2除法不借位，相同位置用异或。

”最后用一个例子演示一下“模2除法”：

![mark](http://mculover666.cn/image/20190808/cvO25i2jnBFf.png?imageslim)

## 关键点2 —— 确定“生成多项式”
生成多项式既确定了如何改造原始数据作为被除数，也确定了除数，还确定了CRC码的位数，是整个CRC码生成过程的关键。

标准的CRC生成多项式如下表：

|名称|生成多项式|简记式|
|:-:|:-:|:-:|
|CRC-4       | x4+x+1                           |       3     |
|CRC-8       |       x8+x5+x4+1                 |      0x31   |                
|CRC-8       |       x8+x2+x1+1                 |      0x07   |               
|CRC-8       |       x8+x6+x4+x3+x2+x1          |      0x5E   |
|CRC-12      |       x12+x11+x3+x+1             |      0x80F  |
|CRC-16      |       x16+x15+x2+1               |      0x8005 |
|CRC16-CCITT |       x16+x12+x5+1               |      0x1021 |
|CRC-32      |       x32+x26+x23+...+x2+x+1     |      0x04C11DB7|         
|CRC-32c     |       x32+x28+x27+...+x8+x6+1    |      0x1EDC6F41|       

# （发送方）如何计算CRC码

假设选择的CRC生成多项式为：
$$
G(X) = X^4 + X^3 + 1
$$
要求：请写出二进制序列10110011的CRC校验码。

计算过程如下：

## 1.将生成多项式转换成二进制数

- 二进制数的总位数 = 最高位的幂次 + 1
- 多项式中只列出二进制值为1的位

根据这个规则可以得到，该生成多项式$G(X)$对应的二进制数总共有 5 位（4+1），其中第 4 位、第 3 位、第 0 位的二进制值为1，其它位均为0，所以该生成多项式的二进制为： `11001`。

## 2.根据生成多项式确定除数

除数就是生成多项式的二进制数，所以除数为：`11001`。

## 3.根据生成多项式确定被除数和CRC码的位数

- CRC码的位数 = 生成多项式的二进制位数 - 1

根据之前转换的生成多项式的二进制位数，CRC码的位数为：`5 - 1 = 4`位。

在原始数据后加上 `CRC码的位数` 个0，作为被除数，即：`101100110000`。

## 4.使用模2除法计算CRC码
除数确定了，被除数也确定了，CRC码的位数也确定了，万事具备，只欠东风，接下来请出我们的“模2除法”，开始计算CRC码：

![mark](http://mculover666.cn/image/20190808/r9MOz2k23mjY.png?imageslim)

由图上的计算过程可知，得到的CRC码为： `0100`。


# (接收方)如何使用CRC码校验数据
接收方接收到原始数据`10110011`和CRC校验码`0100`后，校验数据是否正确的方法如下：

## 1.获取发送方所使用的生成多项式
接收方首先要获取发送方所使用的生成多项式，然后**使用该生成多项式的二进制数确定除数**，即：`11001`。

## 2.根据接收到的数据和CRC码确定被除数

将接收到的数据和CRC码拼接起来，作为被除数，这里为：`101100110100`。

## 3.使用模2除法校验数据正确性
除数确定了，被除数也确定了，接下来再次使用“模2除法”校验：

![mark](http://mculover666.cn/image/20190808/S3BJImRSXHbz.png?imageslim)

由图上的计算过程可知，校验得到的余数为`0`，接收结果：正确！

**一旦接收数据和接收的CRC码中有一位改变，则计算结果余数不为0，校验失败。**

# 使用C语言实现CRC校验
## 算法1 —— 按位校验思想
### CRC4
对于简单的CRC-4，实现代码如下：
```c
#include <stdint.h>

#define CRC4_POLYNOMIAL 0xC8   /* 11011后面补0凑8位数：11011000*/

uint8_t CheckCrc4(uint8_t const message)
{
    uint8_t  remainder;	    //余数
    uint8_t  i = 0;         //循环变量

    /* 初始化，余数=原始数据 */
    remainder = message;

    /* 从最高位开始依次计算  */
    for (i = 0; i < 8; i++)
    {
        if (remainder & 0x80)
        {
            remainder ^= POLYNOMIAL;
        }
        remainder = (remainder << 1);
    }

    /* 返回计算的CRC码 */
    return (remainder >> 4);
}
```
测试代码如下：
```c
#include <stdio.h>
int main(void)
{
    uint8_t dat = 0xB3;
    uint8_t crc = CheckCrc4(dat);
    
    printf("crc = %#x\n", crc);
    if(crc == 0x4)
    {
        printf("ok.\n");
    }
    else
    {
      printf("fail.\n");
    }

    return 0;
}
```
运行结果如下：

![mark](http://mculover666.cn/image/20190809/LcGji03UtDJ9.png?imageslim)

### CRC8
根据一个字节数据的CRC校验实现思想，两个字节或多个字节的数据也是同样的道理，加一层循环就可以了，代码实现如下：
```c
#define CRC8_POLYNOMIAL 0x31

uint8_t CheckCrc8(uint8_t* const message, uint8_t initial_value)
{
    uint8_t  remainder;	    //余数
    uint8_t  i = 0, j = 0;  //循环变量

    /* 初始化 */
    remainder = initial_value;

    for(j = 0; j < 2;j++)
    {
        remainder ^= message[j];

        /* 从最高位开始依次计算  */
        for (i = 0; i < 8; i++)
        {
            if (remainder & 0x80)
            {
                remainder = (remainder << 1)^CRC8_POLYNOMIAL;
            }
            else
            {
                remainder = (remainder << 1);
            }
        }
    }

    /* 返回计算的CRC码 */
    return remainder;
}
```
接下来用最开始在背景中提出的问题进行检，SHT30传感器的数据手册中给出了它计算CRC的生成多项式和初始值，并给出了一个示例，如图：

![mark](http://mculover666.cn/image/20190809/wtLIFxbSLyon.png?imageslim)

接下来我们使用示例测试一下：
```c
int main(void)
{
    char dat[2] = {0xBE,0xEF};
    uint8_t crc = CheckCrc8(dat, 0xFF);
    
    printf("crc = %#x\n", crc);
    if(crc == 0x92)
    {
        printf("ok.\n");
    }
    else
    {
      printf("fail.\n");
    }

    return 0;
}
```
测试结果如下：

![mark](http://mculover666.cn/image/20190809/VXPkJUIakUWH.png?imageslim)

## 算法2 —— 查表思想
### 生成表
首先需要编写一个程序，计算好所有的8位二进制数的CRC校验码，然后将它保存成一个256B大小的数组。

生成数组的代码如下：
```c
int main(void)
{
    uint16_t i = 0;
    uint8_t crc = 0;
    
    for(i = 0; i < 256; i++)
    {
        crc = CheckCrc4(i);
        printf("0x%02x, ", crc);
        if((i+1)%16 == 0)
        {
            printf("\n");
        }
    }

    return 0;
}
```
生成的表如图：

![mark](http://mculover666.cn/image/20190809/ueP7lMrR9bjw.png?imageslim)

### 查找表

将这张表保存为一个数组，然后编写新的校验CRC的程序：
```c
#include <stdio.h>
#include <stdint.h>

uint8_t CRC4_TABLE[256] = {
    //……这里数据太多，省略
};
int main(void)
{
    uint8_t dat = 0xB3;
    uint8_t crc = CRC4_TABLE[dat];      //直接查表
    
    printf("crc = %#x\n", crc);
    if(crc == 0x04)
    {
        printf("ok.\n");
    }
    else
    {
      printf("fail.\n");
    } 
    return 0;
}
```

## 两种C语言实现方法比较

|实现思想|优点|缺点|
|:-:|:-:|:-:|
|按位计算|占用内存空间极小|计算速度慢，数据长度大时计算速度会非常慢|
|查找表|计算速度非常快|占用内存空间大，数据长度大时表会占用非常大的空间|


## 移植第三方库 —— LibCRC
CRC的计算确实是一个非常头疼的事情，所以国外有大神开源了一个库专门用于CRC计算 —— LibCRC。

Libcrc是一个C语言实现的多平台MIT许可CRC库，其官网链接为：[www.libcrc.org](https://www.libcrc.org/)，其Github仓库为：[LibCRC](https://github.com/lammertb/libcrc.git)。

有兴趣的读者可以移植分享一下~

## 使用硬件CRC校验电路
在STM32L4上有一个专门的CRC校验外设，可以直接使用STM32CubeMX激活CRC校验，然后使用HAL调用进行校验，后续在`使用硬件IIC和硬件CRC驱动SHT30`这篇文章中会有介绍，敬请期待~