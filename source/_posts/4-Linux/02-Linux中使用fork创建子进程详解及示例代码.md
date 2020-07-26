---
title: Linux中使用fork创建子进程详解及示例程序
keywords: Linux 子进程 fork
tags: Linux应用开发
categories: Linux
abbrlink: 1603677970
summary: Linux应用开发
date: 2020-03-13 08:00:56
---
# 1. 进程
## 1.1. 什么是进程
当可执行文件开始运行之后，就变为了系统中的一个**进程**，一个程序（可执行文件）运行起来之后可以创建多个进程执行，称之为多进程程序。

每个进程包含有**进程运行环境、内存地址空间、进程ID**、和至少一个被称为线程的执行控制流等资源。

系统中所有的这些进程实体共享计算机系统的 CPU、外设、内存等资源。

## 1.2. 进程的状态
系统中的一个CPU在某一个时刻只能执行一个进程，系统中存在的这些多个进程按照一定的规则轮流交替执行，所以每个进程会产生不同的状态：

- `R`：运行态或者就绪态，一旦等待CPU，立马执行；
- `S`：可中断的睡眠状态（因为等待某种事件的发生而被挂起）；
- `D`：不可中断的睡眠状态（此时不能响应异步信号）；
- `T`：暂停状态；
- `W`：退出状态，进程即将被销毁；
- `Z`：退出状态，进程成为僵尸进程；

![](https://img-blog.csdnimg.cn/20200313185646266.png)

# 2. 编写多进程程序——创建子进程
## 2.1. 头文件
在使用多进程编程的API时，必须首先包含以下头文件：
```c
#include <unistd.h>		//定义了fork函数
```
## 2.2. 创建进程
函数原型如下：
```c
pid_t fork(void);
```
fork函数用来创建子进程，它是<font color="red">**通过复制父进程得到的子进程**</font>，所以：

- 子进程将继承父进程的整个地址空间；
- 子进程只有pid号和父进程不一样；

fork函数会复制出子进程，接着子进程和父进程都处于就绪状态，开始由内核调度执行，所以，被调用的fork函数会<font color="red">**返回两次**</font>：

- 父进程返回一次，返回的 pid_t 类型的值为新创建的子进程的pid；
- 子进程返回一次，返回的 pid_t 类型的值为0；

如果创建失败，父进程会返回-1。

另外，pid_t类型其实就是short类型。


## 2.3. 获取PID的函数
在头文件`<unistd.h>`中定义，原型如下：

- 获取自己的pid
```c
pid_t getpid(void);
```

- 在子进程中获取父进程的pid
```c
pid_t getppid(void);
```
## 2.4. 创建子进程的示例代码

```c

/**
 * @brief  使用fork创建子进程示例程序
 * @author Mculover666
 * @note   创建一个子进程，打印父进程和子进程的pid，并测试n的值
*/

#include <unistd.h>
#include <stdio.h>

int main(void)
{
    pid_t pid;

    int n = 0;  //测试父进程和子进程是否共享一个n

    pid = fork();

    if(pid < 0)
    {
        /* 创建子进程失败 */
        printf("fork fail.\n");
        return -1;
    }
    else if(pid == 0)
    {
        /* 子进程执行的程序 */
        n += 1;
        printf("I am child, my pid = %d, my father pid = %d, n = %d.\n", getpid(), getppid(), n);
        return 0;
    }
    else
    {
        /* 父进程要执行的程序 */
        n += 2;
        printf("I am father, my pid = %d, my child pid = %d, n = %d.\n", getpid(), pid, n);
        return 0;
    }
}

```

编译：

```bash
gcc 02-fork_test.c -o 02-fork_test.o
```

执行：

![](https://img-blog.csdnimg.cn/20200313193621861.png)

由执行结果可以看到，n变量被子进程fork过去之后，在子进程中就有了一个新的变量n，所以父进程执行后n = 2，子进程执行后n = 1，不是同一个n。

## 2.5. 移植到嵌入式Linux开发板上运行测试
① 使用交叉编译工具重新编译程序：
```bash
arm-none-linux-gnueabi-gcc 02-fork_test.c -o 02-fork_test_arm.o
```
![](https://img-blog.csdnimg.cn/20200313194131978.png)

② 基于NFS文件系统，将可执行文件传到开发板上
```bash
cp 02-fork_test_arm.o /nfs_root/
```

③ 开发板上执行该程序测试，结果如下：
![](https://img-blog.csdnimg.cn/20200313194957396.png)

<font color="red">**接收更多精彩文章及资源推送，欢迎订阅我的微信公众号：『mculover666』。**</font>
![](https://img-blog.csdnimg.cn/20200202092055136.png)

