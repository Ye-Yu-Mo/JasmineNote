---
title: Linux高阶IO之select多路转接
date: 2024-09-26 15:55:01
tags: [C++,Linux,网络,IO]
categories: [Linux]
---

## select多路转接

多路转接有三种方案,分别是select,poll和epoll,我们都会讲解和介绍

select的函数原型是这样的

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d30cb4d9529248e0b426295ba8eb33d3.png)

他一共有五个参数,但是可以分为三组

* nfds:需要监视的最大的文件描述符值+1
* readfds:可读文件描述符集合
  writefds:可写文件描述符集合
  exceptfds:异常文件描述符集合
* timeout:设置select等待的时间

就绪事件分为三种,可读事件,可写事件,异常事件

### timeout

select可以帮我们同时等待多个文件描述符,但是如果一直轮询多个描述符是不合理的,我们应当可以设置他阻塞或者轮询

因此timeout可以传入三种数据

* NULL : 表示没有timeout,select一直阻塞等待,直到有了就绪的文件描述符
* 0 : 进检测文件描述符集合的状态,然后立即返回,不会进行任何阻塞等待,继续执行后面的逻辑
* 确定的时间 : 阻塞指定的时间段,没有等到则超时返回

前两个都好理解,第三个的数据类型是`timeval`,要怎么用呢

他的内部有两个成员,分别对应了秒和微秒

长这样

```cpp
struct timeval {
    __time_t      tv_sec;  // 秒
    __suseconds_t tv_usec; // 微秒
};
```

使用的时候需要线对他进行初始化,然后传入select即可

```cpp
struct timeval tv;
tv.tv_sec = 1;
tv.tv_usec = 500;
// 等待1.5秒
```

### fd_set

fd_set是中间三个文件描述符的类型,如果你了解信号,也一定猜到,这个fd_set是一共位图,你可以将你期待监视的文件描述符设置到这个位图中然后传递进去

但是我们不能直接操作这个数据结构,需要使用下面的函数,分别对应着删除\测试\设置\置零

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/127f94b2136540519e516c47c7702782.png)

### 返回值

select的返回值需要区分

* 执行成功 : 返回文件描述符状态改变的个数
* 0 : 阻塞等待超时
* -1 : 出错了,错误原因在errno里,此时三个输出型参数的值是未知的

错误原因可能有

* EBADF : 文件描述符无效或者文件描述符关闭
* EINTR : 调用被信号中断
* EINVAL : n为负值
* ENOMEM : 核心内存不足

### 执行过程

其实最后一个参数timeout也是输出型参数

当你设置阻塞时间为5秒,等了3秒后时间就绪了,函数就返回了,timeout的时间就会变成2秒

fd_set也是这样的,我们把想要监视的文件描述符放在fd_set中,但是他返回的时候,只有就绪的文件描述符返回出来了,如果一开始设置了0,1,2,3,4,之后0和3就绪了,那么我们再拿到对应的fd_set,就只能读到0和3,原来的fd_set就消失了一样

因此如果我们想要循环进行select,每一次都需要重置过参数

### 总结

select能够同时监视多个文件描述符,但是也总是有上限的,上限就是fd_set位图的大小

除此之外,我们不仅需要维护fd_set,还需要额外维护放入fd_set中的数据集,否则我们无法使用FD_ISSET判断文件描述符是否就绪

而且每一次都需要重新逐一加入fd_set,然后再取出最大的文件描述符,作为第一个参数输入

所以这个系统调用是真难用,非常不便,还需要将fd_set从用户态拷贝到内核态,fd_set也不够大

但是这是第一个,有缺陷才是正常的,后面才会越来越好,有了poll和epoll

具体使用select写server的代码可以来我的github仓库来看看

[LearnRep/CppCode at main · Ye-Yu-Mo/LearnRep (github.com)](https://github.com/Ye-Yu-Mo/LearnRep/tree/main/CppCode)
