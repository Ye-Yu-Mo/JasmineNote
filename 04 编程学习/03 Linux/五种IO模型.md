---
title: Linux高级IO之五种IO模型
date: 2024-09-26 12:07:36
tags: [C++,Linux,网络,IO]
categories: [Linux]
---

## Linux高级IO之五种IO模型

### IO的理解

IO模型其实更像是网络部分的延伸和拓展，在学习完计算机网络之后，结合Linux系统，我们就该认识到，计算机网络所做的工作其实是将计算机的功能和性能无限扩大

因为进程间通信的本质没有变，网络将计算机之间的进程通信链接起来，相当于世界的计算机性能也能链接起来，只是受限于网络的性能和复杂度，速度远远达不到计算机总线的速度

进程处理数据自然有数据的处理和交换两个过程

处理部分就是我们所学的各种语言，面向对象思想，算法

而交换则对应了输入和输出两个部分，由于计算机网络的影响，IO也不应当狭义的理解为从内存中IO，而应当也扩展至从网络中IO，虽然设备上，是内存和网卡的区别，但是对于Linux来说他们都是文件，是没什么区别的
我们具体到IO本身上，因为外设的速度上远低于CPU处理的速度的，对于内存和网卡都是这样，因此IO一定是效率很低的事情

在IO时，一共做了两件事情，第一件事情是等待，等待缓冲区的文件，或是网卡上来，或是从内存中来，第二件事情是拷贝，将缓冲区数据拷贝到内存中

因此IO一共分成两个部分，等待+拷贝，其中拷贝使用才是我们希望他做的事情

类似于计算机发展的过程，计算机开始时是等待IO和处理数据两个过程，然后逐步演化为现在的并发情景，将等待IO的时间减少，处理数据的时间增加

我们对IO的期待也是这样，希望等待的时间减少，而IO拷贝的本身时间增加

所谓的IO模型就是提供解决这些问题的方案

### 阻塞式IO

阻塞式IO我们一开始学习语言的时候就在用，例如cin，scanf，都是阻塞式IO

当内核在数据准备好之前，系统调用会阻塞在读取步骤，一直等待，直到缓冲区中有有效数据

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/449ba85c64df481f1c0feff97ed9fcbb.png)

所有的套接字默认都是阻塞式IO，阻塞式IO也是最常见的模型

### 非阻塞IO

非阻塞IO就可以简单理解为不等待的IO，他在读取缓冲区时，发现没有数据，则会直接返回，跟读取出错的情况是很像的

但是我们需要区分这两种情况，利用全局变量errno，当他是读取发现无数据时，则设置为EWOULDBLOCK

非阻塞IO一般是需要反复尝试读写文件描述符，这个过程称之为轮询，对CPU来说是比较大的浪费，一般也只有在特定情况下才会使用

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/da4d067c554348788f280782f4f3ac04.png)

设置IO为非阻塞形式，需要使用系统调用fcntl

函数原型如下:

```cpp
#include <unistd.h>
#include <fcntl.h>
int fcntl(int fd, int cmd, ... /* arg */ );
```

传入的cmd值不同,后面追加的参数也不同,有五种功能

* F_DUPFD:复制一个现有的描述符
* F_GETFD 或 F_SETFD:获得\设置文件描述符标记
* F_GETFL 或 F_SETFL:获得\设置文件状态标记
* F_GETOWN 或 F_SETOWN:获得\设置异步IO
* F_GETLK 或 F_SETLK 或 F_SETLKW:获得\设置记录锁

我们这里是使用第三个功能,获取\设置文件状态标记,将一个文件描述符设置为非阻塞

```cpp
// 将指定fd设置为非阻塞
void SetNonBlock(int fd)
{
    int fl = fcntl(fd, F_GETFL);
    int (fl < 0)
    {
        cerr << "fcntl error" << endl;
        exit(1);
    }
    fcntl(fd, F_SETFL, fl | O_NONBLOCK);
}

int main()
{
    SetNonBlock(0); // 将标准输入设置为非阻塞
    while(true)
    {
        char buffer[1024];
        ssize_t s = read(0, buffer, sizeof(buffer) - 1);
        if(s > 0) // 读取有数据
        {
            buffer[s] = '\0';
            cout << buffer << endl;
        }
        else if (s = 0) // 读到文件结尾
        {
            cout << "file end" << endl;
            break;
        }
        else
        {
            // 出错或者读取无数据
            if(errno == EWOULDBLOCK || errno == EAGAIN)
            {
                // 缓冲区无数据
                cout << "buffer no status" << endl;
                cout << errno << endl;
            }
            else if(errno == EINTR)
            {
                // 信号中断，但是不算read的错误
                cout << "IO interrupted by signal" << endl;
            }
            else
            {
                // 真的读取错误
                cout << "read error" << endl;
                break;
            }
        }
        sleep(1);
    }
    return 0;
}
```

实际上当缓冲区没有数据的时候，甚至是信号中断的时候，你可以在其中写其他的小型业务逻辑，while循环会一直在这里轮询

### 信号驱动式IO

缓冲区是操作系统管理的，因此当缓冲区中有数据时，操作系统是知道的，他就可以使用信号`SIGIO`通知进程进行IO拷贝操作

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9215239bac4be604552daafd06675ad4.png)

这就是信号驱动式IO，我们不用亲自动手去检查IO等待是否完成，而是交由操作系统去，他好了就会通知我们继续完成拷贝，这样我们就有更多的时间去做别的事情

### IO多路转接

IO多路转接是实际应用最为广泛的一种IO模式，因为上面的几种都是一次只管理一个文件描述符，为什么不直接管理一百个文件描述符呢

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/af6b39631f77639e7c149bb7dcead7a0.png)

实际上是可行的，这需要借助select和recvfrom这两个系统调用，第一个是用来查看数据是否准备好，第二个是用来拷贝数据的，这种模式会在下一篇文章具体介绍

### 异步IO

上面所有的IO都是同步IO，说人话就是参与了等待或者拷贝这两个过程的

而异步IO则是都不参与，在拷贝完成之后，系统会通知进程可以处理使用了

与信号驱动IO不同，异步IO不需要参加拷贝的过程，直接进行使用

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2d2391843b5c5d08df7fe408b9e271ce.png)

### 同步和异步

同步和异步是一种思想，是一种消息通信的机制

同步是在调用的时候，就一定要等待这个调用的结果

异步是在调用之后就直接返回，不一定有结果，调用发出之后，被调用者就会通过状态、通知、回调函数来处理这个调用的指令

在实际应用过程中，一定要考虑到业务逻辑适合用哪一种，服务器能否承担得起同步IO的代价
