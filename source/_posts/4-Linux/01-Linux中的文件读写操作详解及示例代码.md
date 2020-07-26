---
title: Linux中的文件读写详解及示例程序
keywords: Linux 文件读写
tags: Linux应用开发
categories: Linux
abbrlink: 917538546
summary: Linux应用开发
date: 2020-03-12 08:00:56
---

# 1. Linux中“一切皆文件”
在Linux系统中，一切皆文件，文件类型根据其表示的意义，分为：

- 普通文件
- 设备文件：代表一个具体的硬件设备
- 管道文件、FIFO文件：具有特殊意义的文件，用于进程间通信；
- 套接字文件：用于网络通信；

所有这些文件都可以用一套API来操作，最基本的四个API是：

- 打开：open
- 读文件：read
- 写文件：write
- 关闭：close

在使用这些API操作文件的时候，需要传入文件标识符 fd（file descriptor），文件标识符的本质就是在进程中代码某一个具体文件的整数，在使用 open 函数打开一个文件时被唯一分配，一般情况下，fd的值如果从0开始分配，如果 fd 为负数，则表示文件打开失败或者操作失败。

需要注意，fd的值在使用open打开文件时被唯一分配，在使用close关闭文件时会被回收。举个例子，比如第一次打开文件时分配的值fd = 0，如果该文件被关闭，下次新打开一个文件时，fd依然为0，但如果之前打开的文件没有被close掉，则下次新打开一个文件时，fd递增为1，一直递增到系统设定的同时打开文件的最大值为止，这个最大值可以使用`ulimit -n`查看，一般为1024。

另外，文件描述符的值0、1、2被系统占用，在桌面Linux系统上表示：

- fd = 0：表示标准输入(stdin)，对应系统的键盘
- fd = 1：表示标准输出(stdout)，对应系统的显示器
- fd = 2：表示标准错误输出(stderr)，对应系统的显示器

而在嵌入式Linux系统中，一般不存在显示器和键盘，都是用串口交互，所以这三个值对应的设备如下：

- fd = 0：表示标准输入(stdin)，对应系统的控制台串口
- fd = 1：表示标准输出(stdout)，对应系统的控制台串口
- fd = 2：表示标准错误输出(stderr)，对应系统的控制台串口


# 2. Linux C库提供的文件操作API
## 2.1. 头文件
在使用文件操作API时，必须首先包含以下头文件：
```c
#include <sys/types.h>		//定义了一些常用数据类型，比如size_t
#include <fcntl.h>			//定义了open、creat等函数，以及表示文件权限的宏定义
#include <unistd.h>			//定义了read、write、close、lseek等函数
#include <errno.h>			//与全局变量errno相关的定义
#include <sys/ioctl.h>		//定义了ioctl函数 
```

## 2.2. open — 打开文件
操作文件之前必须要打开文件，获取文件描述符fd，该函数原型如下：
```c
int open(const char *pathname, int flags);
int open(const char *pathname, int flags,mode_t mode);
```
① 参数含义如下：

- pathname：文件路径+文件名称
- flags：文件打开方式
- mode：打开文件的权限
- 返回值int：打开成功则返回文件描述符，打开失败则返回-1，同时设置全局变量errno的值来表示错误原因；

② flags标志的值可以使用在`<fcntl.h>`的宏定义：

- `O_RDONLY`：只读
- `O_WRONLY`：只写
- `O_RDWR`：可读可写（常用）
- `O_CREAT`：如果要打开的文件不存在，则创建新文件
- `O_EXCL`：如果使用O_CREAT时文件已经存在，则返回错误消息
- `O_TRUNC`：如果文件已经存在，且成功打开，则删除文件中原来的全部数据
- `O_APPEND`：以追加写入方式打开文件，打开之后文件指针指向文件末尾

这些打开方式可以使用`|`操作符，一起使用，比如：
```c
O_RDWR | O_CREAT
```

③ mode的值表示创建新文件时设置的权限，用8进制数来表示，也可以使用<fcntl.h>中定义的宏定义来表示，但是八进制数比较方便。

3bit的八进制数分别对应linux中文件的三个权限：
```
rwxrwxrwx
```
第一个rwx是文件拥有者的权限，第二个rwx是文件拥有者所在用户组的权限，第三个rwx是其它用户组的权限；

每一个rwx都对应一个**8进制数**，比如0x7就表示rwx三项权限全有，0x0就表示rwx三个权限全没有，比如：

- `0700`：所属用户有rwx权限，当前用户组和其他用户组的用户没有任何权限；
- `0664`（常用）：所属用户有rw-权限（-表示没有此项对应权限），当前用户组的其它用户拥有rw-权限，其它用户组的用户只有r--权限。

## 2.3. read — 读取文件
函数原型如下：
```c
ssize_t read(int fd, void *buf, size_t count);
```
函数参数含义如下：

- fd：文件描述符
- buf：用来接收所读数据的缓冲区
- count：请求读取的字节数
- 返回值：读取成功则返回读取的字节数，读取到文件尾则返回0，读取失败则返回-1，同时设置全局变量errno的值来表示错误原因；

## 2.4. write — 写入文件
函数原型如下：
```c
ssize_t write(int fd, const void *buf, size_t count);
```
函数参数含义如下：

- fd：文件描述符
- buf：存放待写入数据的缓冲区
- count：请求写入的字节数
- 返回值：写入成功则返回实际写入的字节数，写入失败则返回-1，同时设置全局变量errno的值来表示错误原因；

## 2.5. close — 关闭文件
API原型如下：
```c
int close(int fd);
```
函数参数fd表示要关闭文件的文件描述符。

如果关闭成功，返回0，否则返回-1，同时设置全局变量 errno 报告具体错误的原因。


# 3. 文件操作示例程序
范例实现功能：

打开当前目录下的text.txt文件，如果不存在则创建，首先写入一个字符串，然后关闭文件，再重新打开读取该文件中的内容并打印。


范例代码：
```c
/**
 * @brief  文件读写API使用示例程序
 * @author Mculover666
 * @note   首先写入文件，接着读取文件
*/

#include <sys/types.h>		//定义了一些常用数据类型，比如size_t
#include <fcntl.h>			//定义了open、creat等函数，以及表示文件权限的宏定义
#include <unistd.h>			//定义了read、write、close、lseek等函数
#include <errno.h>			//与全局变量errno相关的定义
#include <sys/ioctl.h>		//定义了ioctl函数 
#include <stdio.h>

int main(void)
{
    int fd = -1;
    int res = 0;

    char filename[]  = "test.txt";
    char write_dat[] = "Hello World!";
    char read_buf[128] = {0};

    /* 写入文件操作示例 */
    //1. 打开文件
    fd = open(filename, O_RDWR | O_CREAT, 0664);
    if(fd < 0)
    {
        printf("%s file open fail,errno = %d.\r\n", filename, errno);
        return -1;
    }

    //2. 读取内容
    res = write(fd, write_dat, sizeof(write_dat));
    if(res < 0)
    {
        printf("write dat fail,errno = %d.\r\n", errno);
        return -1;
    }
    else
    {
        printf("write %d bytes:%s\r\n", res, write_dat);
    }

    //3. 关闭文件
    close(fd);

    /* 读取文件数据示例 */
    //1. 打开文件
    fd = open(filename, O_RDONLY);
    if(fd < 0)
    {
        printf("%s file open fail,errno = %d.\r\n", filename, errno);
        return -1;
    }

    //2. 写入内容
    res = read(fd, read_buf, sizeof(read_buf));
    if(res < 0)
    {
        printf("read dat fail,errno = %d.\r\n", errno);
        return -1;
    }
    else
    {
        printf("read %d bytes:%s\r\n", res, read_buf);
    }

    //3. 关闭文件
    close(fd);

    return 0;
}
```
编译：
```bash
gcc 01-file_test.c -o 01-file_test.o
```
运行：

![](https://img-blog.csdnimg.cn/20200312193151469.png)

# 4. 移植到嵌入式Linux开发板上运行测试
## 4.1. 优化程序 —— fsync
嵌入式Linux系统通常采用 Flash 存储器， write()有个问题 —— 将数据提交到系统内部缓存之后就返回，所以当它返回之后，嵌入式系统如果突然掉电，则文件数据也会写入失败。

所以，在嵌入式系统中使用这些API时，**应该在write函数之后，用 fsync()函数把修改过的文件数据立马写入闪存中，避免数据丢失**。

fsync()函数的功能原型在<unistd.h>中，原型如下：
```c
int fsync(int fd);
```
fsync()调用后，会直到文件已修改过的数据全部写入磁盘后才返回，成功返回 0，失败返回-1， 同时设置全局变量 errno 报告具体错误的原因。

在示例程序的write操作之后添加代码：
```c
//写入之后立马同步数据
res = fsync(fd);
if(res < 0)
{
    printf("fsync fail,errno = %d.\r\n", errno);
    return -1;
}
```

## 4.2. 移植到开发板上运行处
① 使用交叉编译工具重新编译程序：
```bash
arm-none-linux-gnueabi-gcc 01-file_test.c -o 01-file_test_arm.o
```

![](https://img-blog.csdnimg.cn/20200312195247139.png)

② 基于NFS文件系统，将可执行文件传到开发板上
```bash
cp 01-file_test_arm.o /nfs_root/
```

③ 开发板上执行该程序测试：
```bash
./01-file_test_arm.o
```
开发板上执行结果如下：

![](https://img-blog.csdnimg.cn/20200312195622433.png)

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>

![](https://img-blog.csdnimg.cn/20200202092055136.png)
