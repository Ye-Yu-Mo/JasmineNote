---
title: Linux网络——socket编程与UDP实现服务器与客户机通信
date: 2024-09-11 16:06:38
tags: [C++,Linux,网络]
categories: [Linux]
---

## 端口号

我们说即便是计算机网络，他们之间的通信也仍然是进程间通信

那么要如何在这么多计算机中，找到你想要的那个进程呢

在网络中标识的唯一的计算机使用的是ip地址

在同一台计算机的进程是通过进程id区分的，而要在对方的计算机中按照进程id来找恐怕不是一共好的想法，因为你也不知道对方进程的id是多少，于是就商量（传输层协议）使用port端口来确定进程

因此ip加上port就能确定这个世界上的唯一一台计算机中的唯一一个进程

ip地址是用四个八位二进制数来表示

而port端口号是一共2字节16位整数

端口号用来标识进程，告诉操作系统数据要交给哪一个进程

都已经有了进程id了为什么还要有个端口号呢

这是因为一个端口号只能被一个进程占用，但是一个进程是可以拥有多个端口号的

他们之间并不是完美的1对1关系

## TCP/UDP

我们先简单重新认识一下这两个协议

这两个都是传输层的协议

TCP面向的是有连接，意思是在正式的传递信息之前，需要建立连接，确保是能收到的，就像对暗号一样，土豆土豆我是地瓜

而UDP则是无连接的，相当于直接把数据扔出去

![640?wx_fmt=jpeg](https://i-blog.csdnimg.cn/blog_migrate/8a75a27028eeba55138714bc4e88d5ce.jpeg)

![img](https://img-blog.csdnimg.cn/9465f18bc5f94ee799025ead08541f0d.png)

由此可见，TCP是可靠传输，他规定了一些措施来保证传输的可靠性，例如三次握手四次挥手

> “嗨，我想听一个TCP的笑话。”
>
> “你好，你想听 TCP 的笑话么？”
>
> “嗯，我想听一个 TCP 的笑话。”
>
> “好的，我会给你讲一个TCP 的笑话。”
>
> “好的，我会听一个TCP 的笑话。”
>
> “你准备好听一个TCP 的笑话么？”
>
> “嗯，我准备好听一个TCP 的笑话”
>
> “OK，那我要发 TCP 笑话了。大概有 10 秒，20 个字。”
>
> “嗯，我准备收你那个 10 秒时长，20 个字的笑话了。”
>
> “抱歉，你的链接超时了。你好，你想听 TCP 的笑话么？”

而UDP是完全没有的，因此他是不可靠的

> 我给你们讲个UDP 的笑话吧！ 哈哈哈哈哈哈哈哈哈是不是很好笑

除此之外，TCP由于建立了连接，就可以像水流一样传输数据，是面向字节流的，而UDP则没有，所以UDP是面向数据包的

## 网络字节序

计算机内存分为大端存储和小端存储，大端存储是低地址存高位数据，小端存储是低地址存低位数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/fb4b75d9209940788fff5f839b0e76f8.png)

至今这两个流派也没有分出胜负，但是计算机网络不知道啊，他不知道两个计算机之间是如何存储的，只能硬性规定，网络数据流是按照低地址存高位数据，也就是大端存储

发送方的主机发送时是按照地址从低到高发送的

如果发送方是小端，则先将数据传换成大端

我们可以使用系统调用，将大小端字节序进行交换

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/2c4e9034c9b749c880ff33646064e75d.png)

也就是说小端机器发送时，调用hton，小端机器接收时，调用ntoh

大端直接就原封不动返回了，因此不确定大小端时，调用就对了

## socket的常见API

socket有一个媲美鲁棒性的翻译，套接字，让人看得云里雾里，简直是完美的反自学机制

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/3578319c3bfe40c2a7e668168e6d9fba.png)

套接字的本质就是一个文件描述符，这个东西非常非常非常重要

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/b4f46f44bd654e73890610794d6412e7.jpeg)

socket的第一个参数是标识套接字的类型，AF_INET表示IPv4，也是最常用的

第二个参数表示通信的类型，是使用UDP（SOCK_DGRAM）还是使用TCP（SOCK_STREAM）

bind的作用是将这个进程提供的服务绑定到操作系统中，当外部访问时就能知道这个服务的端口号，当信息发出时，也能携带自身的ip和port

listen和accept是TCP通信中需要用到的，listen用于开始监听是否有请求，accept用于将拿到的请求解析，connect则是连接的请求

接下来我们会使用UDP进行主机和服务器之间的通信

## UDP实现服务器与客户机通信

首先我们需要知道的是客户机和服务器需要的程序是不一样的，因此需要分别实现服务器和客户机的代码

我们需要知道，所谓的服务器和客户机不过都是进程

### 服务器

服务器的代码相对复杂，我们分不同文件来说明

```cpp
// nocopy.hpp
#pragma once
#include <iostream>
class nocopy
{
public:
    nocopy() {}
    nocopy(const nocopy &) = delete;
    const nocopy &operator=(const nocopy &) = delete;
    ~nocopy() {}
};
```

这个类主要是实现一个最简单的单例模式，防止服务器进程被拷贝等内容

```cpp
#pragma once
#include <iostream>
#include <string>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

class InetAddr // 将接收到的网络信息格式化
{
public:
    InetAddr(struct sockaddr_in &addr)
        : _addr(addr)
    {
        _port = ntohs(_addr.sin_port);   // 将网络字节序转换为主机字节序
        _ip = inet_ntoa(_addr.sin_addr); // 将网络字节序的IP转换为点分十进制的字符串
    }
    std::string Ip()
    {
        return _ip;
    }
    uint16_t Port()
    {
        return _port;
    }
    std::string PrintDebug() // 输出调试信息
    {
        std::string info = _ip;
        info += ':';
        info += std::to_string(_port);
        return info;
    }
    ~InetAddr() {}

private:
    std::string _ip;
    uint16_t _port;
    struct sockaddr_in _addr;
};
```

这个类主要是给客户机提供一个存储客户机信息的类，用于返回给客户机的请求

```cpp
#pragma once
#include <iostream>
#include <string>
#include <cstring>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include "nocopy.hpp"
#include "InetAddr.hpp"

const static uint16_t defaultport = 1141; // 默认端口号
const static uint16_t defaultfd = -1;     // 默认socket
const static uint16_t defaultsize = 1024; // 默认缓冲区大小

class UdpServer : public nocopy // 服务器不允许被拷贝，简化的单例模式
{
public:
    UdpServer(uint16_t port = 11451)
        : _port(port)
    {
    }
    void Init()
    {
        // 创建socket文件描述符，对其进行初始化
        _sockfd = socket(AF_INET, SOCK_DGRAM, 0); // 使用IPv4，UDP协议
        if (_sockfd < 0)
        {
            perror("socket创建失败");
            exit(errno);
        }
        std::cout << "socket创建成功,文件描述符为:" << _sockfd << std::endl;

        // 初始化网络信息
        struct sockaddr_in local;
        bzero(&local, sizeof(local));       // memset
        local.sin_family = AF_INET;         // 协议簇
        local.sin_port = htons(_port);      // 端口号
        local.sin_addr.s_addr = INADDR_ANY; // ip地址

        // 绑定到系统内核
        int n = bind(_sockfd, (struct sockaddr *)&local, sizeof(local));
        if (n != 0)
        {
            perror("bind出错");
            exit(errno);
        }
    }
    void Start() // 服务器不退出
    {
        char buffer[1024];
        for (;;)
        {
            struct sockaddr_in peer; // 远程地址信息，因为需要返回请求
            socklen_t len = sizeof(peer);
            ssize_t n = recvfrom(_sockfd, buffer, sizeof(buffer) - 1, 0, (struct sockaddr *)&peer, &len); // 获取从从外部接收的数据
            // 第一个参数是套接字文件描述符，标识接收数据的套接字
            // 第二个参数是接收数据的缓冲区

            if (n > 0) // 正确接收
            {
                InetAddr addr(peer); // 格式化网络信息
                buffer[n] = '\0';    // 手动添加结束
                std::cout << '[' << addr.PrintDebug() << "]# " << buffer << std::endl;
                sendto(_sockfd, buffer, strlen(buffer), 0, (struct sockaddr *)&peer, len);
            }
        }
    }
    ~UdpServer()
    {
    }

private:
    uint16_t _port; // 端口号
    int _sockfd;    // socket文件描述符
};
```

这里就是客户机的主体了，里面的代码和注释写的非常详细，主要思路就是初始化，先听，后回复

```cpp
#include "udpserver.hpp"
#include <memory>

void Usage(std::string proc) // 使用提示
{
    std::cout << "Usage:\n\t" << proc << "local_port\n"
              << std::endl;
}

int main(int argc, char *argv[])
{
    if (argc != 2)
    {
        Usage(argv[0]);
        return 1;
    }
    uint16_t port = std::stoi(argv[1]);
    std::unique_ptr<UdpServer> usvr = std::make_unique<UdpServer>(port);
    usvr->Init();
    usvr->Start();
    return 0;
}
```

这里就是主程序，负责判断用户输入是否正确，调用服务器

### 客户机

```cpp
#include <iostream>
#include <cerrno>
#include <cstring>
#include <string>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>

void Usage (const std::string &proc)
{
    std::cout<<"Usage:\n\t"<<proc<<"server_ip server_port" <<std::endl;
}

int main(int argc, char* argv[])
{
    if(argc!=3)
    {
        Usage(argv[0]);
        return 1;    
    }
    std::string serverip = argv[1];
    uint16_t serverport = std::stoi(argv[2]);

    // 创建socket
    int sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    if(sockfd<0)
    {
        perror("socket创建失败");
        exit(errno);
    }
    std::cout<<"socket创建成功,sockfd:"<<sockfd<<std::endl;

    // 客户机也需要绑定，但是不需要主动调用，客户机会在第一次发送数据时自动绑定数据
    // 填充服务器信息
    struct sockaddr_in server;
    memset(&server, 0,sizeof(server));
    server.sin_family = AF_INET;
    server.sin_port = htons(serverport);
    server.sin_addr.s_addr = inet_addr(serverip.c_str());

    while(true)
    {
        // 发送的数据
        std::string inbuffer;
        std::cout<<"Please Enter# ";
        std::getline(std::cin, inbuffer);
        // 发送消息（请求）
        ssize_t n = sendto(sockfd, inbuffer.c_str(), inbuffer.size(), 0, (struct sockaddr*)&server, sizeof(server));
        // 收消息
        if(n>0)
        {
            char buffer[1024];
            struct sockaddr_in tmp;
            socklen_t len = sizeof(tmp);
            ssize_t m = recvfrom(sockfd, buffer, sizeof(buffer)-1, 0, (struct sockaddr*)&tmp, &len);
            if(m>0)
            {
                buffer[m] = '\0';
                std::cout<<"server echo# "<<buffer<<std::endl;
            }
            else
                break;
        }
        else
            break;
    }   
    close(sockfd);
    return 0;
}
```

客户机是先发出请求，再接收请求

### 运行效果如下

![image.png](https://s2.loli.net/2024/09/11/oa3491yvcMzUdkp.png)

使用`netstat -tuln`可以查到服务器的ip地址和端口，当然这里只适合使用本机调试
