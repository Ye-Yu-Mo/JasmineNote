---
title: Linux进程间通信——SystemV共享内存
date: 2024-08-18 17:23:13
tags: [Linux,进程]
categories: [Linux]
---

## System V共享内存

System V是一种进程间通信的标准，在这种标准之下，本地通信的方式一般由三种，内存共享、消息队列、信号量

这里我们主要介绍共享内存

### 创建内存共享

![image.png](https://s2.loli.net/2024/08/18/xMnWzmNIFHjypkb.png)

这是内存共享的系统调用

第一个参数是共享内存在内核中的标识，类似于文件标识符的地位

第二个参数是创建共享内存的大小

第三个参数有两个选项IPC_CREAT和IPC_EXCL

如果只有第一个，则使用共享内存，不存在则创建后继续使用

第二个不能单独使用，两个相或之后表示创建共享内存后使用，存在则出错返回

第三种方法主要是为了确保所使用的共享一定是新鲜的

创建成功会返回共享内存的标识符，失败返回-1，错误码表示错误原因

共享内存存在内存的共享区中，可以有很多个，是被操作系统描述组织管理的

对于第一个参数，key是需要确定唯一的，类似于手机号，你自己有，别人不能再有一模一样的，并且别人想要打给你也必须输入一模一样的

我们可以使用ftok函数进行生成，只要两个参数相同，则能生成相同的key，类似于哈希函数

![image.png](https://s2.loli.net/2024/08/18/gE9kQjxinOPb6FK.png)

需要注意的是，共享内存在进程退出时是不会主动释放的，需要用户手动释放，除非系统内核关闭

### 释放共享内存

#### 命令行

```shell
ipcs
```

使用这个命令可以查看当前资源，三种资源全包含

```shell
ipcs -m
```

查看共享内存

```shell
ipcrm -m shmid
```

删除对应shmid的共享内存

![image.png](https://s2.loli.net/2024/08/18/mQbeW5ZKf1GcNSp.png)

#### shmctl

这个函数主要是用于控制共享内存，有设置获取删除三种功能

![image.png](https://s2.loli.net/2024/08/18/9KqHVALoEI81h4W.png)

shmid是共享内存id，cmd表示操作，buf是用来获取共享内存的属性的，各种属性都存在该结构体里

获取用IPC_STAT，设置用IPC_SET，删除用IPC_RMID

#### shmat

at是attach是关联的意思，因为共享的物理内存是属于操作系统的，我们需要将共享的内存挂载到进程地址空间的共享区，就要用到shamat

![image.png](https://s2.loli.net/2024/08/18/dbRyfo8Utur3AnK.png)

第一个参数表示要将哪个共享内存挂载

第二个参数表示要挂在到哪个地址，可以使用nullptr自动

第三个参数表示挂载的形式，都还是写，一般默认为0，保证读写

挂载成功后会返回地址，失败返回-1

挂接后我们就可以直接访问这个共享内存了

#### shmdt

dt是detach，分离的意思，也就是将共享内存删除，也可以说成去关联

![image.png](https://s2.loli.net/2024/08/18/whF2YEOgTPCmKAr.png)
