---
title: 基于muduo库实现protobuf协议的通信详解
date: 2024-09-29 14:02:49
tags: [C++,Linux,框架]
categories: [框架]
---

在之前我们单纯使用muduo实现的时候,并没有考虑到tcp粘包之类的问题,只是进行一个返回

但是在实际应用过程中绝对不能这么草率,需要实现一个应用层协议来解决这些问题

包括序列化和反序列化,粘包

## muduo定制的protobuf协议

在muduo源文件的example中有实现protobuf的客户端和服务端

我们可以先大致学习一下这里是怎么实现的,然后仿照使用一下

![image.png](https://s2.loli.net/2024/09/29/KGh2Ti1sZcFfRCP.png)

可以看到这里消息回调函数,这里面用的是ProtoBufCodec的onMessage函数

dispatcher是一个分发器,codec_是一个协议处理器

初始化里面是有一个注册消息回调函数,后面会有大用

![image.png](https://s2.loli.net/2024/09/29/bdWB1ZTsOiDfxhG.png)

而在这个类中,我们可以看到onMessage和send,其实就是针对protobuf处理的方法,也就是收到消息之后被调用的方法

在具体看这个onMessage处理函数之前,我们先看这个协议的结构是什么样的

![image.png](https://s2.loli.net/2024/09/29/47ZUgnIRPluCyme.png)

有了这样的协议之后,就可以解决粘包问题

具体的函数调用逻辑是这样的

![null.png](https://s2.loli.net/2024/09/29/iJTcp1RDlNus28y.png)

业务操作逻辑是这样的

![null _1_.png](https://s2.loli.net/2024/09/29/SOZ4GHs3JdFNvKy.png)

## proto文件

```protobuf
syntax = "proto3";

package Xu;

message TranslateRequest {
    string msg = 1;
};

message TranslateResponse {
    string msg = 1;
};

message AddRequest {
    int32 num1 = 1;
    int32 num2 = 2;
};

message AddResponse{
    int32 result =1;
};
```

使用`protoc --cpp_out=./ Request.proto`可以自动生成源文件和头文件

## 服务端代码编写

```cpp
#include "protobuf/codec.h"
#include "protobuf/dispatcher.h"

#include "muduo/include/muduo/base/Logging.h"
#include "muduo/include/muduo/base/Mutex.h"
#include "muduo/include/muduo/net/EventLoop.h"
#include "muduo/include/muduo/net/TcpServer.h"

#include "Request.pb.h"
#include "TransLate.hpp"
#include "../logs/Xulog.h"

#include <iostream>
#include <unordered_map>
#include <unistd.h>

class Server
{
public:
    using MessagePtr = std::shared_ptr<google::protobuf::Message>;
    using TranslateRequestPtr = std::shared_ptr<Xu::TranslateRequest>;
    using TranslateResponsePtr = std::shared_ptr<Xu::TranslateResponse>;
    using AddRequestPtr = std::shared_ptr<Xu::AddRequest>;
    using AddResponsePtr = std::shared_ptr<Xu::AddResponse>;

    Server(int port)
        : _server(&_baseloop, muduo::net::InetAddress("0.0.0.0", port),
                  "Server", muduo::net::TcpServer::kReusePort),
          _dispatcher(std::bind(&Server::onUnknowMessage, this,
                                std::placeholders::_1, std::placeholders::_2, std::placeholders::_3)),
          _codec(std::bind(&ProtobufDispatcher::onProtobufMessage, &_dispatcher,
                           std::placeholders::_1, std::placeholders::_2, std::placeholders::_3))
    {
        // 注册业务请求处理函数
        _dispatcher.registerMessageCallback<Xu::TranslateRequest>(std::bind(&Server::onTranslate, this, std::placeholders::_1,
                                                                            std::placeholders::_2, std::placeholders::_3));
        _dispatcher.registerMessageCallback<Xu::AddRequest>(std::bind(&Server::onAdd, this, std::placeholders::_1,
                                                                      std::placeholders::_2, std::placeholders::_3));
        _server.setMessageCallback(std::bind(&ProtobufCodec::onMessage, &_codec, std::placeholders::_1,
                                             std::placeholders::_2, std::placeholders::_3));
        _server.setConnectionCallback(std::bind(&Server::onConnection, this, std::placeholders::_1));
    }
    void start()
    {
        _server.start();
        _baseloop.loop();
    }

private:
    std::string translate(const std::string &str)
    {
        if ((str[0] >= 'a' && str[0] <= 'z') || (str[0] >= 'A' && str[0] <= 'z'))
            return Translate(str, "en", "zh");
        return Translate(str, "zh", "en");
    }

    void onAdd(const muduo::net::TcpConnectionPtr &conn, const AddRequestPtr message, muduo::Timestamp)
    {
        // 提取Message中的有效消息
        int num1 = message->num1();
        int num2 = message->num2();
        // 进行计算得到结果
        int ans = num1 + num2;
        // 组织并发送protobuf的相应
        Xu::AddResponse resp;
        resp.set_result(ans);
        _codec.send(conn, resp);
    }
    void onTranslate(const muduo::net::TcpConnectionPtr &conn, const TranslateRequestPtr message, muduo::Timestamp)
    {
        // 提取Message中的有效消息
        std::string req_msg = message->msg();
        // 进行翻译得到结果
        std::string rsp_msg = translate(req_msg);
        // 组织并发送protobuf的相应
        Xu::TranslateResponse resp;
        resp.set_msg(rsp_msg);
        _codec.send(conn, resp);
    }
    void onUnknowMessage(const muduo::net::TcpConnectionPtr &conn, const Server::MessagePtr message, muduo::Timestamp)
    {
        INFO("onUnknowMessage: %s", message->GetTypeName());
        conn->shutdown();
    }
    void onConnection(const muduo::net::TcpConnectionPtr &conn)
    {
        if (conn->connected())

            INFO("连接建立成功!");
        else
            INFO("连接关闭!");
    }

private:
    muduo::net::EventLoop _baseloop;
    muduo::net::TcpServer _server;  // 服务器对象
    ProtobufDispatcher _dispatcher; // 请求分发器对象 -> 注册请求处理函数
    ProtobufCodec _codec;           // protobuf协议处理器 -> 对收到的请求数据进行protobuf协议处理
};

int main()
{
    Server server(8085);
    server.start();
    return 0;
}
```

## 客户端代码编写

```cpp
#include "protobuf/dispatcher.h"
#include "protobuf/codec.h"

#include "muduo/include/muduo/base/Mutex.h"
#include "muduo/include/muduo/base/Logging.h"
#include "muduo/include/muduo/net/EventLoop.h"
#include "muduo/include/muduo/net/TcpClient.h"
#include "muduo/include/muduo/net/EventLoopThread.h"
#include "muduo/include/muduo/base/CountDownLatch.h"

#include "Request.pb.h"
#include "../logs/Xulog.h"

#include <iostream>
#include <unistd.h>

class Client
{
public:
    using MessagePtr = std::shared_ptr<google::protobuf::Message>;
    using TranslateResponsePtr = std::shared_ptr<Xu::TranslateResponse>;
    using AddResponsePtr = std::shared_ptr<Xu::AddResponse>;
    Client(const std::string &sip, int sport)
        : _latch(1), _client(_loopthread.startLoop(), muduo::net::InetAddress(sip, sport), "Client"),
          _dispatcher(std::bind(&Client::onUnknowMessage, this,
                                std::placeholders::_1, std::placeholders::_2, std::placeholders::_3)),
          _codec(std::bind(&ProtobufDispatcher::onProtobufMessage, &_dispatcher,
                           std::placeholders::_1, std::placeholders::_2, std::placeholders::_3))
    {
        _dispatcher.registerMessageCallback<Xu::TranslateResponse>(std::bind(&Client::onTranslate, this, std::placeholders::_1,
                                                                             std::placeholders::_2, std::placeholders::_3));
        _dispatcher.registerMessageCallback<Xu::AddResponse>(std::bind(&Client::onAdd, this, std::placeholders::_1,
                                                                       std::placeholders::_2, std::placeholders::_3));
        _client.setMessageCallback(std::bind(&ProtobufCodec::onMessage, &_codec, std::placeholders::_1,
                                             std::placeholders::_2, std::placeholders::_3));
        _client.setConnectionCallback(std::bind(&Client::onConnection, this, std::placeholders::_1));
    }
    // 连接服务器 阻塞等待连接建立成功
    void connect()
    {
        _client.connect();
        _latch.wait(); // 阻塞等待,直到连接建立成功
    }
    void Translate(const std::string &str)
    {
        Xu::TranslateRequest req;
        req.set_msg(str);

        send(&req);
    }
    void Add(int num1, int num2)
    {
        Xu::AddRequest req;
        req.set_num1(num1);
        req.set_num2(num2);

        send(&req);
    }

private:
    bool send(const google::protobuf::Message *msg)
    {
        if (_conn->connected())
        {
            _codec.send(_conn, *msg);
            return true;
        }
        return false;
    }

    void onAdd(const muduo::net::TcpConnectionPtr &conn, const AddResponsePtr message, muduo::Timestamp)
    {
        INFO("加法结果是: %d", message->result());
    }
    void onTranslate(const muduo::net::TcpConnectionPtr &conn, const TranslateResponsePtr message, muduo::Timestamp)
    {
        INFO("翻译结果是: %s", message->msg());
    }
    void onConnection(const muduo::net::TcpConnectionPtr &conn)
    {
        if (conn->connected())
        {
            _latch.countDown();
            _conn = conn;
        }
        else
        {
            _conn.reset();
        }
    }

    void onUnknowMessage(const muduo::net::TcpConnectionPtr &conn, const Client::MessagePtr message, muduo::Timestamp)
    {
        INFO("onUnknowMessage: %s", message->GetTypeName());
        conn->shutdown();
    }

private:
    muduo::CountDownLatch _latch;            // 实现同步
    muduo::net::EventLoopThread _loopthread; // 异步循环处理线程
    muduo::net::TcpConnectionPtr _conn;      // 客户端连接
    muduo::net::TcpClient _client;           // 客户端句柄
    ProtobufDispatcher _dispatcher;          // 请求分发器
    ProtobufCodec _codec;                    // 协议处理器
};

int main()
{
    Client client("127.0.0.1", 8085);
    client.connect();
    client.Translate("Hello");
    client.Add(1, 6);
    sleep(1);
    return 0;
}
```

![null _2_.png](https://s2.loli.net/2024/09/30/gAdWq3phaPY4Mr7.png)
