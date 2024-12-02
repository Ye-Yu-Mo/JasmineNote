---
title: Linux网络——守护进程、会话、进程组
date: 2024-09-15 21:10:07
tags: [C++,Linux,网络]
categories: [Linux]
---

## 会话

会话是session，代表的是客户端与服务器的一次交互过程，我们可以简单理解为，当我们打开一个终端，在用户登录时，就是创建了一个会话

一般来说会话都与各自的终端关联，这个会话中的第一个进程称之为首进程，也称为进程组长

一个会话中可能有很多进程，我们称之为进程组，为了表示也就有了进程组ID：PGID，会话ID：SIG

![image.png](https://s2.loli.net/2024/09/15/BfwdWZ7Ibsz2PVx.png)

使用setsid系统调用可以创建新会话

![image.png](https://s2.loli.net/2024/09/15/sLS4yTVQ1GfeEgn.png)

需要注意的是，创建会话的进程不能是组长进程

进程组的主要特征就是信号可以发给一个进程组中的所有进程，可以进行作业控制

PGID就是这个进程组的唯一标识，其实也就是进程组组长的id

也就是要求一个进程只能当一个组的组长，否则id不就重复了嘛

但是说进程组长挂了，进程组中只要还有一个进程存在，整个进程组还在

## 守护进程

守护进程也叫做精灵进程，Deamon，是一种运行在后台的进程，所谓前台后台可以简单理解为能否直接收到键盘指令的进程

守护进程独立于终端，并且周期性的执行某种任务或者等待处理某些发生的事件

当Linux系统启动时，或者bash启动时，会启动很多进程服务，例如ssh连接，或者一些代理服务，在进程终止或者系统终止时服务才停止

这种服务就是没有受到终端的控制，没有直接和用户交互

这些服务进程我们称之为守护进程

### 编写守护进程的注意事项

要创建会话就需要创建一个子进程，让他来调用setsid就可以了

之前我们学过，当文件的读端被关闭时，写端继续写就会导致进程挂掉，SIGPIPE

TCP是面向字节流的，他的本质其实也就是一个大号的文件，因此也会遇到这样的情况

这时候我们不想让进程挂掉，因为当很多客户机连接到一个服务器时，一个进程出问题，让整个服务器都受到影响岂不是得不偿失，因此就需要忽略SIGPIPE

除此之外，还需要将整个进程的工作目录换到根目录，主要是为了保证进程不会被意外终止

还有一点是，需要将三个文件标识符定向到 `/dev/null`，因为不需要进行输入输出操作，只是提供服务，因此这三个东西就没啥用了

`/dev/null`可以理解为输入输出的垃圾桶，Minecraft的岩浆

往里面可以丢任何的东西，但是却拿不出来任何东西

### 编写样例

```cpp
// Deamon.hpp
#pragma once

#include <iostream>
#include <cstdlib>
#include <signal.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/stat.h>

const char *root = "/";             // 根目录
const char *dev_null = "/dev/null"; // 销毁

void Deamon(bool ischdir, bool isclose)
{
    // 忽略会引起进程异常退出的信号
    signal(SIGCHLD, SIG_IGN);
    signal(SIGPIPE < SIG_IGN);

    // 自己不能是组长
    if (fork() > 0) // 创建一个子进程，如果自己是父进程则退出，子进程则继续走下去
        exit(0);

    // 创建新的会话，从此都是子进程
    setsid();

    // 每个进程都有一个CWD，当前工作目录，更改为根目录
    if (ischdir)
        chdir(root);

    // 关闭标准输入、输出、错误
    if (isclose)
    {
        close(0);
        close(1);
        close(2);
    }
    else
    {
        int fd = open(dev_null, ORDWR);
        if (fd > 0)
        {
            dup2(fd, 0);
            dup2(fd, 1);
            dup2(fd, 2);
            close(fd);
        }
    }
}
```
