---
title: muduo网络库介绍
date: 2024-09-27 19:41:22
tags: [C++,Linux,框架]
categories: [框架]
---

## Muduo

Muduo是陈硕大佬开发的,一个基于非阻塞IO和事件驱动的C++高并发网络编程库

这是一个基于主从Reactor模型的网络编程库,线程模型是one loop per thread

意思是一个线程只能有一个事件循环(event loop), 用于响应计时器和事件

一个文件描述符只能由一个线程进行读写,也就是一个Tcp连接,必须归属于一个EventLoop管理

基本逻辑是这样的

![image.png](https://s2.loli.net/2024/09/27/QsPaboWA4lGUkYr.png)

所谓的主从Reactor就是,在主线程中有一个主Reactor进行事件触发,而在其他其他线程中就是普通的Reactor进行事件触发

在主线程中主要任务就是监控新连接的到来,保证尽可能高效的获取新建连接,再按照负载均衡的方式分发到普通Reactor的线程中,对对应的IO事件进行监控

主从Reactor必然是一个多执行流的并发模式,也就是one thread one loop

### Server常见接口

#### TcpServer类

这个类用来生成服务器对象

==构造函数是这样的==

```cpp
TcpServer(EventLoop *loop,
          const InetAddress &listenAddr,
          const string &nameArg,
          Option option = kNoReusePort);
```

第一个参数loop对象在下面来介绍

第二个参数是一个地址信息,结构是这样的

```cpp
class InetAddress : public muduo::copyable
{
public:
    InetAddress(StringArg ip, uint16_t port, bool ipv6 = false);
};
```

包含了ip和端口两个信息,是服务器需要监听和绑定的地址

第三个参数是字符串的名称

最后一个参数是一个选项,是否启用端口重用的功能

```cpp
enum Option
{
    kNoReusePort,
    kReusePort,
};
```

我们直到当请求释放连接时,会有一段time wait状态,这段时间内同一个端口号是无法再次被绑定使用的

打开端口重用就可以解除这个限制,继续使用这个端口号

==setThreadNum()==

```cpp
void setThreadNum(int numThreads);
```

这个成员函数就是用于设置从属Reactor线程的数量的

==Start()==

```cpp
void start();
```

启动事件监听

==setConnectionCallback()==

这是一个回调函数,需要我们自己来编写

```cpp
typedef std::function<void(const TcpConnectionPtr &)> ConnectionCallback;
void setConnectionCallback(const ConnectionCallback &cb)
{
    connectionCallback_ = cb;
}
```

当连接建立成功时,会调用这个回调接口,完成我们的功能

==setMessageCallback()==

这也是一个回调函数,是用于业务处理的回调函数

```cpp
typedef std::function<void(const TcpConnectionPtr &,
                           Buffer *,
                           Timestamp)>
    MessageCallback;
void setMessageCallback(const MessageCallback &cb)
{
    messageCallback_ = cb;
}
```

当收到连接的新的消息的时候,调用的函数

#### EventLoop类

这个类主要是用于事件监控和业务处理,在构造TcpServer之前,就需要构造这个EventLoop对象,最重要的就是loop成员函数

#### TcpConnection类

connected()和disconnect()是用于查看连接状态的

send()是用于发送数据的

### 服务器搭建

```cpp
#include "muduo/include/muduo/net/TcpServer.h"
#include "muduo/include/muduo/net/EventLoop.h"
#include "muduo/include/muduo/net/TcpConnection.h"
#include "../logs/Xulog.h"
#include "TransLate.hpp"
#include <iostream>
#include <functional>
#include <unordered_map>

class TranslateServer
{
public:
    TranslateServer(int port) : _server(&_baseloop,
                                        muduo::net::InetAddress("0.0.0.0", port),
                                        "TranslateServer",
                                        muduo::net::TcpServer::kReusePort)
    {
        // 参数绑定
        auto func_1 = std::bind(&TranslateServer::onConnection, this, std::placeholders::_1);
        auto func_2 = std::bind(&TranslateServer::onMessage, this, std::placeholders::_1,
                                std::placeholders::_2, std::placeholders::_3);
        // 设置回调函数
        _server.setConnectionCallback(func_1);
        _server.setMessageCallback(func_2);
    }

    // 启动服务器
    void start()
    {
        _server.start();  // 开始事件监听
        _baseloop.loop(); // 开始事件监控,死循环阻塞接口
    }

private:
    // 建立连接或关闭之后的回调函数
    void onConnection(const muduo::net::TcpConnectionPtr &conn)
    {
        if (conn->connected())
        {
            INFO("新连接建立成功!");
        }
        else
        {
            INFO("连接关闭!");
        }
    }
    std::string translate(const std::string &str)
    {
        return Translate(str, "en", "zh");
    }
    // 收到请求时的回调函数
    void onMessage(const muduo::net::TcpConnectionPtr &conn, muduo::net::Buffer *buf, muduo::Timestamp)
    {
        // 调用translate接口进行翻译,向客户端返回结果
        std::string str = buf->retrieveAllAsString();
        std::string resp = translate(str);
        conn->send(resp);
    }

private:
    muduo::net::EventLoop _baseloop;
    muduo::net::TcpServer _server;
};

int main()
{
    TranslateServer server(8085);
    server.start();
    return 0;
}
```

这里我们使用了百度翻译的api,可以将英文翻译成中文

```cpp
#include <iostream>
#include <string>
#include <curl/curl.h>
#include <cstdlib>
#include <cstring>
#include <openssl/evp.h>
#include <openssl/err.h>
#include <jsoncpp/json/json.h>
#include <sstream>
#include <iomanip>

static size_t WriteCallback(void *contents, size_t size, size_t nmemb, std::string *userp)
{
    size_t total_size = size * nmemb;
    userp->append(static_cast<char *>(contents), total_size);
    return total_size;
}

std::string generateSign(const std::string &appid, const std::string &q, const std::string &salt, const std::string &secret_key)
{
    std::string sign = appid + q + salt + secret_key;
    unsigned char md[EVP_MAX_MD_SIZE];
    unsigned int md_len;

    EVP_MD_CTX *ctx = EVP_MD_CTX_new();
    if (!ctx)
    {
        std::cerr << "Failed to create context for MD5." << std::endl;
        return "";
    }

    if (EVP_DigestInit_ex(ctx, EVP_md5(), nullptr) != 1 ||
        EVP_DigestUpdate(ctx, sign.c_str(), sign.length()) != 1 ||
        EVP_DigestFinal_ex(ctx, md, &md_len) != 1)
    {
        std::cerr << "Error generating MD5 digest." << std::endl;
        EVP_MD_CTX_free(ctx);
        return "";
    }

    EVP_MD_CTX_free(ctx);

    char buf[33] = {0};
    for (unsigned int i = 0; i < md_len; ++i)
    {
        sprintf(buf + i * 2, "%02x", md[i]);
    }
    return std::string(buf);
}

std::string parseJsonResponse(const std::string &response)
{
    Json::CharReaderBuilder reader;
    Json::Value jsonData;
    std::string errs;

    std::istringstream s(response);
    if (Json::parseFromStream(reader, s, &jsonData, &errs))
    {
        std::string from = jsonData["from"].asString();
        std::string to = jsonData["to"].asString();
        std::string translatedText = jsonData["trans_result"][0]["dst"].asString();

        return translatedText;
    }
    else
    {
        std::cerr << "Failed to parse JSON: " << errs << std::endl;
        exit(0);
    }
}

std::string urlEncode(const std::string &value)
{
    std::ostringstream escaped;
    escaped << std::hex << std::setfill('0');
    for (const char c : value)
    {
        if (isalnum(c) || c == '-' || c == '_' || c == '.' || c == '~')
        {
            escaped << c;
        }
        else
        {
            escaped << '%' << std::setw(2) << static_cast<int>(static_cast<unsigned char>(c));
        }
    }
    return escaped.str();
}

std::string _translate(const std::string &appid, const std::string &secret_key, const std::string &q, const std::string &from, const std::string &to)
{
    char salt[60];
    sprintf(salt, "%d", rand());

    std::string sign = generateSign(appid, q, salt, secret_key);
    std::string q_encoded = urlEncode(q);
    std::string myurl = "http://api.fanyi.baidu.com/api/trans/vip/translate?appid=" + appid + "&q=" + q_encoded +
                        "&from=" + from + "&to=" + to + "&salt=" + salt + "&sign=" + sign;
    CURL *curl = curl_easy_init();
    std::string response_string;

    if (curl)
    {
        curl_easy_setopt(curl, CURLOPT_URL, myurl.c_str());
        curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, WriteCallback);
        curl_easy_setopt(curl, CURLOPT_WRITEDATA, &response_string);
        CURLcode res = curl_easy_perform(curl);
        if (res != CURLE_OK)
        {
            std::cerr << "curl_easy_perform() failed: " << curl_easy_strerror(res) << std::endl;
        }
        curl_easy_cleanup(curl);
    }
    return response_string;
}

enum
{
    EnToZh,
    ZhToEn,
    Exit
};

std::string Translate(std::string text_to_translate, std::string from, std::string to)
{
    std::string appid = "";         // 替换为你的 App ID
    std::string secret_key = ""; // 替换为你的密钥
    std::string response = _translate(appid, secret_key, text_to_translate, from, to);
    return parseJsonResponse(response);
}
```

### Client常见接口

#### TcpClient类

==构造函数==

```cpp
TcpClient(EventLoop *loop,
          const InetAddress &serverAddr,
          const string &nameArg);
```

这里需要传入一个Eventloop,我们的epoll监控就在这里面

第二个参数是要连接的服务器的地址信息

第三个参数是名称

==connect()==

这个成员函数是用来连接服务器的

==disconnect()==

停止连接

==stop()==

停止客户端运行

==connection()==

```cpp
TcpConnectionPtr connection() const
{
    MutexLockGuard lock(mutex_);
    return connection_;
}
```

这个成员函数是用来获取客户端通信连接的Connection对象的接口

这是一个非阻塞接口,调用之后直接返回,连接不一定建立完成,不能直接发送数据

如果需要在连接完成之后再做操作,需要调用内置的`CountDownLatch`类中的wait()成员函数进行同步控制

==setConnectionCallback()==

```cpp
void setConnectionCallback(ConnectionCallback cb)
{
    connectionCallback_ = std::move(cb);
}
```

这是连接服务器成功时的回调函数

==setMessageCallback()==

```cpp
void setMessageCallback(MessageCallback cb)
{
    messageCallback_ = std::move(cb);
}
```

这是收到服务器发送的消息时的回调函数

### 客户端搭建

```cpp
#include "muduo/include/muduo/net/TcpClient.h"
#include "muduo/include/muduo/net/EventLoopThread.h"
#include "muduo/include/muduo/net/TcpConnection.h"
#include "muduo/include/muduo/base/CountDownLatch.h"
#include "../logs/Xulog.h"
#include <iostream>
#include <functional>

class TranslateClient
{
public:
    TranslateClient(const std::string &sip, int sport) : _lanch(1), // 设置阻塞
                                                         _client(_loopthread.startLoop(), muduo::net::InetAddress(sip, sport), "TranslateClient")

    {
        auto func_1 = std::bind(&TranslateClient::onConnection, this, std::placeholders::_1);
        auto func_2 = std::bind(&TranslateClient::onMessage, this, std::placeholders::_1,
                                std::placeholders::_2, std::placeholders::_3);
        _client.setConnectionCallback(func_1);
        _client.setMessageCallback(func_2);
    }
    // 连接服务器 阻塞等待连接建立成功
    void connect()
    {
        _client.connect();
        _lanch.wait(); // 阻塞等待,直到连接建立成功
    }
    bool send(const std::string &msg)
    {
        if (_conn->connected())
        {
            _conn->send(msg);
            return true;
        }
        return false;
    }

private:
    // 建立连接或关闭之后的回调函数 唤醒阻塞
    void onConnection(const muduo::net::TcpConnectionPtr &conn)
    {
        if (conn->connected())
        {
            _lanch.countDown(); // 唤醒主线程阻塞
            _conn = conn;
            std::cout << "连接建立成功!" << std::endl;
        }
        else
        {
            _conn.reset();
            std::cout << "连接已经断开!" << std::endl;
        }
    }

    // 收到消息时的回调函数
    void onMessage(const muduo::net::TcpConnectionPtr &conn, muduo::net::Buffer *buf, muduo::Timestamp)
    {
        std::cout << "翻译完成啦,结果是: " << buf->retrieveAllAsString() << std::endl;
    }

private:
    muduo::CountDownLatch _lanch;
    muduo::net::EventLoopThread _loopthread;
    muduo::net::TcpClient _client;
    muduo::net::TcpConnectionPtr _conn;
};

int main()
{
    TranslateClient client("127.0.0.1", 8085);
    client.connect();
    while (true)
    {
        std::string buffer;
        std::cin >> buffer;
        client.send(buffer);
    }
    return 0;
}
```

