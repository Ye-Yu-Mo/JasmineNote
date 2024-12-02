---
title: Linux网络——手撕TCP服务器，制定应用层协议，实现网络版计算器
date: 2024-09-16 19:21:47
tags: [C++,Linux,网络]
categories: [Linux]
---

## TCP连接

我们这里不去直接讲TCP连接的各种复杂的内容，只是先简单认识，先熟悉再逐步理解原理

前面我们说TCP是面向字节流的，并且需要先连接才能进行传输数据

那么这对于计算机来说意味着什么

### 序列化

首先计算机网络中的各类设备本身就是没办法做到完全相同的，计算机各种型号，操作系统的各种型号

我们想发送的数据不一定是单纯的数值、字符，完全有可能是结构体，是类，我们想要传输这些内容，需要做的就是讲这些数据和结构体转换为字节流，这个过程我们称之为序列化

那么反过来，从字节流解释出来各种数据和结构体的过程就是反序列化

而且这个序列化和反序列化的过程是必须要相同的规则，不然就像字符串加密解密一样，没办法得到你想要的数据，这个相同的规则规矩，我们就称之为协议

这个序列化和反序列化可以自己定制协议，我们后面也会手动实现

也有常见的序列化方案，都是现成写好的，例如json、protobuf、xml，我们也会进行使用

除此之外，既然是面向字节流的，我们如何知道一段数据的开始和结尾呢，就像水流一样，如果不做限制是很难分清的，因此我们还需要在数据的开始和结尾做标识

## 网络版本的计算器

```cpp
// Daemon.hpp
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

void Daemon(bool ischdir, bool isclose)
{
    // 忽略会引起进程异常退出的信号
    signal(SIGCHLD, SIG_IGN);
    signal(SIGPIPE, SIG_IGN);

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
        int fd = open(dev_null, O_RDWR);
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

```cpp
// Socket.hpp
#pragma once

#include <iostream>
#include <string>
#include <cstring>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>

#define Convert(addrptr) ((struct sockaddr *)addrptr)

namespace NetWork
{
    const static int defaultsockfd = -1;
    const int backlog = 5;

    enum
    {
        SocketError = 1,
        BindError,
        ListenError
    };
    class Socket
    {
    public:
        virtual ~Socket() {}
        virtual void CreateSocketOrDie() = 0;
        virtual void BindSocketOrDie(uint16_t port) = 0;
        virtual void ListenSocketOrDie(int backlog) = 0;
        virtual Socket *AcceptConnection(std::string *peerip, uint16_t *peerport) = 0;
        virtual bool ConnectServer(std::string &serverip, uint16_t serverport) = 0;
        virtual int GetSockFd() = 0;
        virtual void SetSockFd(int sockfd) = 0;
        virtual void CloseSockFd() = 0;
        virtual bool Recv(std::string *buffer, int size) = 0;
        virtual void Send(std::string &send_str) = 0;

    public:
        void BuildListenSocketMethod(uint16_t port, int backlog)
        {
            CreateSocketOrDie();
            BindSocketOrDie(port);
            ListenSocketOrDie(backlog);
        }
        bool BuildConnectSockedMethod(std::string &serverip, uint16_t serverport)
        {
            CreateSocketOrDie();
            return ConnectServer(serverip, serverport);
        }
        void BuildNormalSockMethod(int sockfd)
        {
            SetSockFd(sockfd);
        }
    };

    class TcpSocket : public Socket
    {
    public:
        TcpSocket(int sockfd = defaultsockfd)
            : _sockfd(sockfd)
        {
        }
        ~TcpSocket() {}
        void CreateSocketOrDie() override
        {
            _sockfd = ::socket(AF_INET, SOCK_STREAM, 0);
            if (_sockfd < 0)
                exit(SocketError);
        }
        void BindSocketOrDie(uint16_t port) override
        {
            struct sockaddr_in local;
            memset(&local, 0, sizeof(local));
            local.sin_family = AF_INET;
            local.sin_addr.s_addr = INADDR_ANY;
            local.sin_port = htons(port);

            int n = ::bind(_sockfd, Convert(&local), sizeof(local));
            if (n < 0)
                exit(BindError);
        }
        void ListenSocketOrDie(int backlog) override
        {
            int n = ::listen(_sockfd, backlog);
            if (n < 0)
                exit(ListenError);
        }
        Socket *AcceptConnection(std::string *peerip, uint16_t *peerport) override
        {
            struct sockaddr_in peer;
            socklen_t len = sizeof(peer);
            int newsockfd = ::accept(_sockfd, Convert(&peer), &len);
            if (newsockfd < 0)
                return nullptr;
            *peerport = ntohs(peer.sin_port);
            *peerip = inet_ntoa(peer.sin_addr);
            Socket *s = new TcpSocket(newsockfd);
            return s;
        }
        bool ConnectServer(std::string &serverip, uint16_t serverport) override
        {
            struct sockaddr_in server;
            memset(&server, 0, sizeof(server));
            server.sin_family = AF_INET;
            server.sin_addr.s_addr = inet_addr(serverip.c_str());
            server.sin_port = htons(serverport);

            int n = ::connect(_sockfd, Convert(&server), sizeof(server));
            if (n == 0)
                return true;
            else
                return false;
        }
        int GetSockFd() override
        {
            return _sockfd;
        }
        void SetSockFd(int sockfd) override
        {
            _sockfd = sockfd;
        }
        void CloseSockFd() override
        {
            if (_sockfd > defaultsockfd)
                ::close(_sockfd);
        }
        bool Recv(std::string *buffer, int size) override
        {
            char inbuffer[size];
            ssize_t n = recv(_sockfd, inbuffer, size - 1, 0);
            if (n > 0)
            {
                inbuffer[n] = 0;
                *buffer += inbuffer;
                return true;
            }
            return false;
        }
        void Send(std::string &send_str) override
        {
            send(_sockfd, send_str.c_str(), send_str.size(), 0);
        }

    private:
        int _sockfd;
    };
}
```

```cpp
// Protocol.hpp
#pragma once

#include <iostream>
#include <memory>
#include <jsoncpp/json/json.h>

// 定制协议
namespace Protocol
{
    const std::string ProtSep = " ";
    const std::string LineBreakSep = "\n";

    // 对报文进行打包
    // "len\nx op y\n" 这是一个完整报文，以'\n'为分界
    std::string Encode(const std::string &message)
    {
        std::string len = std::to_string(message.size());
        std::string package = len + LineBreakSep + message + LineBreakSep;
        return package;
    }

    // 对报文解包，判断报文的完整性，正确处理有边界的报文
    bool Decode(std::string &package, std::string *message)
    {
        auto pos = package.find(LineBreakSep);
        if (pos == std::string::npos)
            return false;
        std::string lens = package.substr(0, pos);
        int messagelen = std::stoi(lens);
        int total = lens.size() + messagelen + 2 * LineBreakSep.size();
        if (package.size() < total)
            return false;
        *message = package.substr(pos + LineBreakSep.size(), messagelen);
        package.erase(0, total);
        return true;
    }

    class Request
    {
    public:
        Request()
            : _data_x(0), _data_y(0), _oper(0)
        {
        }
        Request(int x, int y, char op)
            : _data_x(x), _data_y(y), _oper(op)
        {
        }
        void Inc()
        {
            _data_x++;
            _data_y++;
        }
        // 结构化数据->字符串
        // 利用条件编译控制自行定义和Json定义
        bool Serialize(std::string *out)
        {
#ifdef SelfDefine
            *out = std::to_string(_data_x) + ProtSep + _oper + ProtSep + std::to_string(_data_y);
            return true;
#else
            Json::Value root;
            root["datax"] = _data_x;
            root["datay"] = _data_y;
            root["oper"] = _oper;
            Json::FastWriter writer;
            *out = writer.write(root);
            return true;
#endif
        }
        // 字符串->结构化数据
        bool Deserialize(std::string &in)
        {
#ifdef SelfDefine
            auto left = in.find(ProtSep);
            if (left == std::string::npos)
                return false;
            auto right = in.rfind(ProtSep);
            if (right == std::string::npos)
                return false;

            _data_x = std::stoi(in.substr(0, left));
            _data_y = std::stoi(in.substr(right + ProtSep.size()));
            std::string oper = in.substr(left + ProtSep.size(), right - (left + ProtSep.size()));
            if (oper.size() != 1)
                retrun false;
            _oper = oper[0];
            return true;
#else
            Json::Value root;
            Json::Reader reader;
            bool res = reader.parse(in, root);
            if (res)
            {
                _data_x = root["datax"].asInt();
                _data_y = root["datay"].asInt();
                _oper = root["oper"].asInt();
            }
            return res;
#endif
        }
        int GetX() { return _data_x; }
        int GetY() { return _data_y; }
        char GetOper() { return _oper; }

    private:
        // 运算格式 _data_x _oper _data_y
        int _data_x;
        int _data_y;
        char _oper;
    };

    class Response
    {
    public:
        Response()
            : _result(0), _code(0)
        {
        }
        Response(int result, int code)
            : _result(result), _code(code)
        {
        }
        bool Serialize(std::string *out)
        {
#ifdef SelfDefine
            *out = std::to_string(_result) + ProtSep + std::to_string(_code);
            return true;
#else
            Json::Value root;
            root["result"] = _result;
            root["code"] = _code;
            Json::FastWriter writer;
            *out = writer.write(root);
            return true;
#endif
        }
        bool Deserialize(std::string &in)
        {
#ifdef SelfDefine
            auto pos = in.find(ProtSep);
            if (pos == std::string::npos)
                return false;
            _result = std::stoi(in.substr(0, pos));
            _code = std::stoi(in.substr(pos + ProtSep.size()));
            return true;
#else
            Json::Value root;
            Json::Reader reader;
            bool res = reader.parse(in, root);
            if (res)
            {
                _result = root["result"].asInt();
                _code = root["code"].asInt();
            }
            return res;
#endif
        }
        void SetResult(int res) { _result = res; }
        void SetCode(int code) { _code = code; }
        int GetResult() { return _result; }
        int GetCode() { return _code; }

    private:
        // 返回格式 "len\nresult code\n"
        int _result; // 运算结果
        int _code;   // 结果码
    };
    // 简单工厂模式
    class Factory
    {
    public:
        std::shared_ptr<Request> BuildRequest()
        {
            return std::make_shared<Request>();
        }
        std::shared_ptr<Request> BuildRequest(int x, int y, char op)
        {
            return std::make_shared<Request>(x, y, op);
        }
        std::shared_ptr<Response> BuildResponse()
        {
            return std::make_shared<Response>();
        }
        std::shared_ptr<Response> BuildResponse(int result, int code)
        {
            return std::make_shared<Response>(result, code);
        }
    };
}
```

```cpp
#pragma once

#include <iostream>
#include <memory>
#include "Protocol.hpp"

// 业务代码
namespace CalculateNS
{
    enum
    {
        Success = 0,
        DivZeroErr,
        ModZeroErr,
        UnknowOper
    };
    class Calculate
    {
    public:
        Calculate() {}
        std::shared_ptr<Protocol::Response> Cal(std::shared_ptr<Protocol::Request> req)
        {
            std::shared_ptr<Protocol::Response> resp = factory.BuildResponse();
            resp->SetCode(Success);
            switch (req->GetOper())
            {
            case '+':
                resp->SetResult(req->GetX() + req->GetY());
                break;
            case '-':
                resp->SetResult(req->GetX() - req->GetY());
                break;
            case '*':
                resp->SetResult(req->GetX() * req->GetY());
                break;
            case '/':
            {
                if (req->GetY() == 0)
                    resp->SetCode(DivZeroErr);
                else
                    resp->SetResult(req->GetX() / req->GetY());
            }
            break;
            case '%':
            {
                if (req->GetY() == 0)
                    resp->SetCode(ModZeroErr);
                else
                    resp->SetResult(req->GetX() % req->GetY());
            }
            break;
            default:
                resp->SetCode(UnknowOper);
                break;
            }
            return resp;
        }
        ~Calculate() {}

    private:
        Protocol::Factory factory;
    };
}
```

```cpp
#include "Protocol.hpp"
#include "Socket.hpp"
#include <iostream>
#include <string>
#include <ctime>
#include <cstdlib>
#include <unistd.h>

using namespace Protocol;

int main(int argc, char *argv[])
{
    if (argc != 3)
    {
        std::cout << "Usage :\n\t" << argv[0] << "serverip sercerport" << std::endl;
        return 0;
    }
    std::string serverip = argv[1];
    uint16_t serverport = std::stoi(argv[2]);

    NetWork::Socket *conn = new NetWork::TcpSocket();
    if (!conn->BuildConnectSockedMethod(serverip, serverport))
    {
        std::cerr << "connect " << serverip << ":" << serverport << " faild" << std::endl;
    }
    std::cerr << "connect " << serverip << ":" << serverport << " success" << std::endl;
    std::unique_ptr<Factory> factory = std::make_unique<Factory>();
    srand(time(nullptr));
    const std::string opers = "+-*/%__";
    while (true)
    {
        int x = rand() % 114;
        usleep(rand() % 2000);
        int y = rand() % 514;
        char oper = opers[rand() % opers.size()];
        std::shared_ptr<Request> req = factory->BuildRequest(x, y, oper);

        std::string requeststr;
        req->Serialize(&requeststr);
        std::cout << requeststr << std::endl;
        std::string testreq = requeststr;
        testreq += " ";
        testreq += "= ";

        requeststr = Encode(requeststr);
        std::cout << requeststr << std::endl;

        conn->Send(requeststr);
        std::string responsestr;
        while (true)
        {
            if (!conn->Recv(&responsestr, 1024))
                break;
            std::string response;
            if (!Decode(responsestr, &response))
                continue;
            auto resp = factory->BuildResponse();
            resp->Deserialize(response);

            std::cout << testreq << resp->GetResult() << "[" << resp->GetCode() << "]" << std::endl;
            break;
        }
        sleep(1);
    }
    conn->CloseSockFd();
    return 0;
}
```

```cpp
#pragma once

#include "Socket.hpp"
#include <iostream>
#include <pthread.h>
#include <functional>

using func_t = std::function<std::string(std::string &, bool *error_code)>;

class TcpServer;

class ThreadData
{
public:
    ThreadData(TcpServer *tcp_this, NetWork::Socket *sockp)
        : _this(tcp_this), _sockp(sockp)
    {
    }

public:
    TcpServer *_this;
    NetWork::Socket *_sockp;
};

class TcpServer
{
public:
    TcpServer(uint16_t port, func_t handler_request)
        : _port(port), _listensocket(new NetWork::TcpSocket()), _handler_request(handler_request)
    {
    }
    static void *ThreadRun(void *args)
    {
        pthread_detach(pthread_self());
        ThreadData *td = static_cast<ThreadData *>(args);

        std::string inbufferstream;
        while (true)
        {
            bool ok = true;
            // 读取报文
            if (!td->_sockp->Recv(&inbufferstream, 1024))
                ;
            break;
            // 处理报文
            std::string send_string = td->_this->_handler_request(inbufferstream, &ok);
            if (ok)
            {
                // 发送数据
                if (!send_string.empty())
                {
                    td->_sockp->Send(send_string);
                }
            }
            else
            {
                break;
            }
        }
        td->_sockp->CloseSocket();
        delete td->_sockp;
        delete td;
        return nullptr;
    }
    void Loop()
    {
        while (true)
        {
            std::string peerip;
            uint16_t peerport;
            NetWork::Socket *newsock = _listensocket->AcceptConnection(&peerip, &peerport);
            if (newsock == nullptr)
                continue;
            std::cout << "get a new connection,sockfd:" << newsock->GetSockFd() << "client info:" << peerip << ":" << peerport << std::endl;
            pthread_t tid;
            ThreadData *td = new ThreadData(this, newsock);
            pthread_create(&tid, nullptr, ThreadRun, td);
        }
    }
    ~TcpServer()
    {
        delete _listensocket;
    }

private:
    int _port;
    NetWork::Socket *_listensocket;

public:
    func_t _handler_request;
};
```

```cpp
#include "Protocol.hpp"
#include "TcpServer.hpp"
#include "Calculate.hpp"
#include "Daemon.hpp"
#include <iostream>
#include <memory>
#include <unistd.h>

using namespace NetWork;
using namespace Protocol;
using namespace CalculateNS;

// 网络负责IO
// HandlerRequest负责字节流数据解析和调用业务
std::string HandlerRequest(std::string &inbufferstream, bool *error_code)
{
    *error_code = true;
    // 业务对象
    Calculate calculte;

    // 工厂对象，构建请求对象
    std::unique_ptr<Factory> factory = std::make_unique<Factory>();
    auto req = factory->BuildRequest();

    // 分析字节流，查看是否报文是否完整
    std::string total_resp_string;
    std::string message;
    while (Decode(inbufferstream, &message))
    {
        std::cout << message << "----messgae" << std::endl;
        // 读取完整报文，进行反序列化
        if (!req->Deserialize(message))
        {
            std::cout << "Deserialize error" << std::endl;
            *error_code = false;
            return std::string();
        }
        std::cout << "Deserialize success" << std::endl;
        // 处理业务
        auto resp = calculte.Cal(req);
        // 序列化响应结果
        std::string send_string;
        resp->Serialize(&send_string);
        // 构建响应字符串
        send_string = Encode(send_string);
        // 发送
        total_resp_string += send_string;
    }
    return total_resp_string;
}

int main(int argc, char *argv[])
{
    // if (argc != 2)
    // {
    //     std::cout << "Usage :\n\t" << argv[0] << " port" << std::endl;
    //     return 0;
    // }
    // uint16_t localport = std::stoi(argv[1]);

    uint16_t localport = 8888;
    // Fork 子进程
    pid_t pid = fork();
    if (pid < 0)
    {
        std::cerr << "Fork failed." << std::endl;
        return 1;
    }
    else if (pid > 0)
    {
        // 父进程退出
        return 0;
    }
    Daemon(false, false);
    std::unique_ptr<TcpServer> svr(new TcpServer(localport, HandlerRequest));
    svr->Loop();
    return 0;
}
```

