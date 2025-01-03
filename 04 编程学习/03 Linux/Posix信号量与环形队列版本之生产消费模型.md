---
title: Linux多线程——POSIX信号量与环形队列版本之生产消费模型
date: 2024-09-07 15:29:45
tags:
  - Linux
  - 多线程
categories:
  - Linux
---

## POSIX信号量

POSIX信号量和System V信号量是不同的标准

但是实现的功能是一样的，都是为了解决同步的问题

我们说信号量指的就是资源的数量

在生产者与消费者模型里面，生产者与消费者所认为的资源是不同的

生产者认为空间是资源，因为每次都要往里面放东西，空间会变少

消费者认为数据是资源，因为每次都会拿东西，数据会变少

### POSIX的操作

#### 初始化

```cpp
#include<semaphore.h>
int sem_init(sem_t *sem, int pshared, unsigned int value);
```

第一个是声明的信号量，第二个是选项，0表示线程间共享，非0表示进程间共享，第三个表示信号量的初始值

#### 销毁

```cpp
int sem_destory(sem_t *sem);
```

#### 等待信号量（申请资源）

申请资源就是P操作

```cpp
int sem_wait(sem_t* sem);
```

#### 发布信号量（放下资源）

发布信号量就是V操作

```cpp
int sem_post(sem_t* sem);
```

## 环形队列之生产消费模型

上一篇文章的生产消费模型是基于阻塞队列的，而且只用了互斥同步锁的内容，我们感觉效率其实不高，因为需要频繁考虑互斥同步问题

如果使用一个环形队列，那就只用在生产者和消费者碰头的时候考虑同步互斥问题

起始时刻，队列为空

生产者的信号量为n

消费者的信号量为0

如果队列为空，需要让生产者先运行

如果队列为满，需要让消费者先运行

除此之外，还需要互斥锁，因为消费者之间和生产者之间也需要互斥访问
