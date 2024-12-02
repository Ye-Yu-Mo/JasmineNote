---
title: Linux高级IO之poll与epoll
date: 2024-09-27 19:35:26
tags: [C++,Linux,网络,IO]
categories: [Linux]
---

poll和epoll都是多路转接的调用，但是epoll实在过于优秀了，一般也都是用epoll的，除此之外还着使用时还蕴含着Reactor设计模式的思想

## poll

poll几乎是解决了select的痛点问题的，就像c和c with class一样

### 使用

poll的函数原型和数据结构长这样

```cpp
#include <poll.h>

int poll(struct pollfd *fd, nfds_t nfds, int tiemout);

struct pollfd{
    int fd;			// 文件描述符
    short events;	// 时间类型
    short revents;	// 实际发生的时间
};
```

+ fds：只想一个pollfd结构体数组的指针，每一个结构体都在监视一个文件描述符
+ nfds：文件描述符的数量
+ timeout：与select相同，只是直接使用int作为类型，单位是毫秒

## epoll

epoll是为了处理大量句柄而做了改进的poll，在实际中运用的最多的也是这个

### 系统调用

![](https://cdn.nlark.com/yuque/0/2024/png/43731355/1727425342914-1f5d63b0-ed4b-4afb-8542-e4ba1ef87c4f.png)

这个系统调用是用于创建一个epoll模型的

![](https://cdn.nlark.com/yuque/0/2024/png/43731355/1727425401966-8aab1a81-78cb-4123-9c2c-51a28aa9a2a0.png)

这个系统调用是用于设置监听的文件描述符和事件

最后一个epoll_event是这样的

```cpp
struct epoll_event{
    uint32_t events;
    epoll_data_t data;
};
```

events表示事件发生时要做的操作，也就是需要监听的事件

data则是监听对应的文件描述符

events也是一个位图，使用宏来表示事件

+ EPOLLIN：表示对应的文件描述符可以读
+ EPOLLOUT：表示对应的文件描述符可以写
+ EPOLLRI：表示对应的文件描述符有紧急数据可读
+ EPOLLRR：表示文件描述符出错
+ EPOLLHUP：表示文件描述符被挂断
+ EPOLLET：表示设为边缘出发模式
+ EPOLLONESHOT：只监听一次事件，如果需要继续监听，需要重新加入EPOLL队列中

![](https://cdn.nlark.com/yuque/0/2024/png/43731355/1727427142637-963f5b6b-9180-446b-81f5-3b230ddbf5bb.png)

epoll_wait是用于等待是否就绪

下面是一个简单的epoll示例

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <sys/epoll.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>

#define MAX_EVENTS 10
#define TIMEOUT 5000  // 5秒

int main() {
    int epfd = epoll_create1(0);
    if (epfd == -1) {
        perror("epoll_create1");
        exit(EXIT_FAILURE);
    }

    // 假设我们要监听标准输入（fd = 0）
    struct epoll_event ev;
    ev.events = EPOLLIN;  // 监听读事件
    ev.data.fd = STDIN_FILENO;

    if (epoll_ctl(epfd, EPOLL_CTL_ADD, STDIN_FILENO, &ev) == -1) {
        perror("epoll_ctl: stdin");
        close(epfd);
        exit(EXIT_FAILURE);
    }

    struct epoll_event events[MAX_EVENTS];
    int nfds;

    while (1) {
        nfds = epoll_wait(epfd, events, MAX_EVENTS, TIMEOUT);
        if (nfds == -1) {
            if (errno == EINTR)
                continue;  // 被信号中断，重新调用
            perror("epoll_wait");
            break;
        }

        if (nfds == 0) {
            printf("等待超时，没有事件发生。\n");
            continue;
        }

        for (int i = 0; i < nfds; ++i) {
            if (events[i].data.fd == STDIN_FILENO) {
                if (events[i].events & EPOLLIN) {
                    char buffer[1024];
                    ssize_t count = read(STDIN_FILENO, buffer, sizeof(buffer) - 1);
                    if (count == -1) {
                        perror("read");
                    } else if (count == 0) {
                        printf("标准输入关闭。\n");
                        close(epfd);
                        exit(EXIT_SUCCESS);
                    } else {
                        buffer[count] = '\0';
                        printf("读取到数据: %s", buffer);
                    }
                }
            }
            // 这里可以处理其他文件描述符的事件
        }
    }

    close(epfd);
    return 0;
}
```

## epoll的工作原理

epoll的内核中使用了两个数据结构来管理文件描述符，分别是一个红黑树，一个队列

### 红黑树

红黑树主要用于存放所有正在监视的文件描述符，当我们使用epoll_ctl设置关心的事件，就是在这个红黑树上进行增删改

### 队列

队列存放的是就绪事件的文件描述符，也就是epoll_wait所等待的就绪的文件描述符

用户使用的门槛其实降低了很多，只需要设置监视，然后获取结果，不需要对fd和event进行管理

每一个epoll对象都有一个独立的eventpoll结构体，用来存放通过ctl向epoll对象添加的事件

事件会放在红黑树，因此插入的时间是O(lg n)

添加到epoll到事件会与设备建立回调关系，当事件发生，调用这个回调方法

这个回调方法会将发生的事件放在rdlist双联白哦

而每一个事件都对应着一个epitem结构体

里面是这样的

```cpp
struct epitem{
    struct rb_node rbn; 		// 红黑树节点
    struct list_head rdllink; 	// 双向链表节点
    struct epoll_filefd ffd; 	// 事件句柄信息
    struct eventpoll *ep;		// 只想其所属的eventpoll对象
    struct epoll_event event;	// 期待发生的事件类型
};
```

当epoll_wait检查事件是否发生时，只需要查看epitem的rdlist是否为空就绪，这个事件复杂度就只有O(1)

## epoll的工作模式

epoll有两种工作模式

一种是水平触发LT，另一种是边缘触发ET

### 水平触发

水平触发就可以理解为阻塞的触发，如果事件发生了，那就会一直进行等待并且通知

epoll的默认工作模式就是水平触发，当epoll检测到事件就绪时，可以不立即进行处理，或者仅处理一部分，一直到缓冲区的所有数据都被处理完，才不会立即返回，支持阻塞和非阻塞

但是这样做的代价很高，因为不能处理返回其他事情

### 边缘触发

边缘触发就是类似于非阻塞的情况，事件发生时，只通知一次，爱拿不拿，不保证数据依然还在，你只有一次处理机会

ET的性能会比LT高很多，而Nginx默认采用的就是ET模式

但是ET只支持非阻塞

## Reactor设计模式

reactor是一种用于处理事件的设计模式，核心思想就是将事件和处理分开

用一个事件的循环来监控多个IO事件，当事件发生时，reactor会调用相应的处理器（回调函数）来处理这些事件

### 工作原理

一般就是分成四个逻辑

1. 注册事件，将事件源和处理器注册到事件循环中
2. 等待事件，事件循环持续监控事件源，等待事件发生
3. 分发事件，事件发生，事件循环调用处理器
4. 处理事件，处理器执行具体到业务逻辑

应用场景，一般就是高并发服务器（HTTP服务器），图形用户界面，网络通信

### epoll Reactor设计模式的简单示例

```cpp
#include <iostream>
#include <sys/epoll.h>
#include <unistd.h>
#include <fcntl.h>
#include <cstring>
#include <unordered_map>

const int MAX_EVENTS = 10;

// 事件处理器基类
class EventHandler {
public:
    virtual void handleEvent(uint32_t events) = 0;
};

// Reactor类
class Reactor {
public:
    Reactor() {
        epoll_fd = epoll_create1(0);
        if (epoll_fd == -1) {
            std::cerr << "Failed to create epoll file descriptor" << std::endl;
            exit(EXIT_FAILURE);
        }
    }

    ~Reactor() {
        close(epoll_fd);
    }

    // 注册事件处理器
    void registerHandler(int fd, EventHandler* handler, uint32_t events) {
        struct epoll_event ev;
        ev.events = events;
        ev.data.fd = fd;

        if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, fd, &ev) == -1) {
            std::cerr << "Failed to add file descriptor to epoll" << std::endl;
            exit(EXIT_FAILURE);
        }

        handlers[fd] = handler;
    }

    // 运行事件循环
    void run() {
        struct epoll_event events[MAX_EVENTS];
        while (true) {
            int n = epoll_wait(epoll_fd, events, MAX_EVENTS, -1);
            if (n == -1) {
                std::cerr << "epoll_wait failed" << std::endl;
                exit(EXIT_FAILURE);
            }

            for (int i = 0; i < n; ++i) {
                int fd = events[i].data.fd;
                uint32_t event_types = events[i].events;
                if (handlers.count(fd)) {
                    handlers[fd]->handleEvent(event_types);
                }
            }
        }
    }

private:
    int epoll_fd;
    std::unordered_map<int, EventHandler*> handlers;
};

// 自定义事件处理器
class MyEventHandler : public EventHandler {
public:
    void handleEvent(uint32_t events) override {
        if (events & EPOLLIN) {
            char buffer[1024];
            ssize_t count = read(STDIN_FILENO, buffer, sizeof(buffer));
            if (count == -1) {
                std::cerr << "Read error" << std::endl;
            } else if (count == 0) {
                std::cout << "EOF" << std::endl;
            } else {
                std::cout << "Read: " << std::string(buffer, count) << std::endl;
            }
        }
    }
};

int main() {
    Reactor reactor;
    MyEventHandler handler;

    // 将标准输入(STDIN_FILENO)注册到epoll中，监听读事件(EPOLLIN)
    reactor.registerHandler(STDIN_FILENO, &handler, EPOLLIN);

    // 开始事件循环
    reactor.run();

    return 0;
}
```

