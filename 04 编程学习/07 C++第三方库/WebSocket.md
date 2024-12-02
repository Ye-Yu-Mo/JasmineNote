---
title: WebSocket协议介绍与C++实现
date: 2024-11-13 20:39:26
tags: [C++]
categories: [C++]
---

WebSocket 是一种计算机网络协议，提供了全双工、低延迟的双向通信。它广泛用于实时数据传输场景，比如即时消息、在线游戏、实时股票行情、实时通知等应用中。与传统的HTTP协议不同，WebSocket支持持久连接，允许客户端和服务器保持一个持久的连接，通过该连接进行双向数据通信。

## WebSocket协议概述

WebSocket协议建立在HTTP协议之上，首先通过一个HTTP请求来进行握手。一旦握手完成，客户端和服务器之间的连接会被“升级”到WebSocket协议，之后通信不再依赖HTTP，而是直接使用WebSocket协议。

### WebSocket握手过程

WebSocket协议的握手过程是基于HTTP协议的，过程如下：

1. **客户端发起请求**：
   客户端发起一个HTTP请求，要求服务器将协议升级为WebSocket。请求的格式如下：

   ```HTTP
   GET /chat HTTP/1.1
   Host: example.com
   Upgrade: websocket
   Connection: Upgrade
   Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
   Sec-WebSocket-Version: 13
   ```

   关键字段说明：
   - `Upgrade: websocket`：告知服务器，客户端希望将协议从HTTP升级为WebSocket。
   - `Sec-WebSocket-Key`：一个随机生成的密钥，用于防止跨站请求伪造（CSRF）攻击。
   - `Sec-WebSocket-Version`：指定WebSocket的协议版本。

2. **服务器响应**：
   如果服务器支持WebSocket协议，并且能够处理该请求，则会返回一个HTTP 101响应，表示协议升级为WebSocket：

   ```HTTP
   HTTP/1.1 101 Switching Protocols
   Upgrade: websocket
   Connection: Upgrade
   Sec-WebSocket-Accept: x3JJHMbDL1EzLkh9tA6c5m9l5mQ==
   ```

   关键字段说明：
   - `Sec-WebSocket-Accept`：服务器生成的密钥，它是通过`Sec-WebSocket-Key`与一个固定的GUID字符串经过SHA-1算法计算得到的。客户端会用这个值进行验证，从而确保服务器的身份。

3. **连接建立**：
   握手成功后，客户端和服务器就可以通过WebSocket协议进行数据传输了。此时，HTTP协议的头部就不再使用，WebSocket协议开始工作。

### WebSocket协议的工作原理

WebSocket连接一旦建立，客户端和服务器之间就会建立一条持久的双向通信通道，任何一方都可以随时发送消息给对方。WebSocket支持文本数据、二进制数据（如图像、音频、视频等），以及ping/pong控制消息。

- **文本消息**：客户端和服务器可以发送UTF-8编码的文本消息。
- **二进制消息**：WebSocket也支持传输二进制数据，包括`Blob`、`ArrayBuffer`等。
- **ping/pong消息**：WebSocket协议内置了心跳机制，用于检测连接是否仍然活跃。客户端或服务器可以发送ping消息，接收方必须回复pong消息。

## C++实现WebSocket

在C++中，我们可以使用`websocketpp`或`Boost.Beast`等库来实现WebSocket的客户端和服务器功能。为了简化实现，本文将以`websocketpp`库为例，展示如何构建一个WebSocket服务器和客户端。

### 服务器端实现

首先，安装`websocketpp`库并链接Boost库。`websocketpp`提供了简单的API来处理WebSocket的连接和消息传递。下面是一个简单的WebSocket服务器的代码实现。

#### 服务器代码：

```cpp
#include <iostream>
#include <websocketpp/config/asio_no_tls.hpp>
#include <websocketpp/server.hpp>

typedef websocketpp::server<websocketpp::config::asio> server;

server echo_server;

// 连接打开时的回调
void on_open(server* s, websocketpp::connection_hdl hdl) {
    std::cout << "New connection established!" << std::endl;
}

// 处理接收到的消息并回传的回调
void on_message(server* s, websocketpp::connection_hdl hdl, server::message_ptr msg) {
    std::cout << "Received: " << msg->get_payload() << std::endl;
    s->send(hdl, msg->get_payload(), websocketpp::frame::opcode::text);  // 回传消息
}

int main() {
    try {
        // 初始化服务器
        echo_server.init_asio();

        // 设置事件回调
        echo_server.set_open_handler(&on_open);
        echo_server.set_message_handler(&on_message);

        // 启动监听
        echo_server.listen(9002);
        echo_server.start_accept();

        std::cout << "Server is running on ws://localhost:9002" << std::endl;

        // 进入事件循环，等待和处理客户端连接
        echo_server.run();
    } catch (websocketpp::exception const& e) {
        std::cerr << "Error: " << e.what() << std::endl;
    }
}
```

### 关键技术点：

- **事件驱动**：`websocketpp`基于事件驱动架构，使用`set_open_handler`设置连接打开时的回调，使用`set_message_handler`处理接收到的消息。通过这些回调函数，服务器可以与客户端进行交互。
- **异步IO**：`echo_server.init_asio()`启用了`Asio`库来进行异步IO操作，这使得服务器能够高效地处理多个并发连接。

### 客户端实现

客户端通过WebSocket协议与服务器进行通信。下面是一个基本的WebSocket客户端实现。

#### 客户端代码：

```cpp
#include <iostream>
#include <websocketpp/config/asio_no_tls.hpp>
#include <websocketpp/client.hpp>

typedef websocketpp::client<websocketpp::config::asio> client;

client ws_client;

// 连接建立时的回调
void on_open(client* c, websocketpp::connection_hdl hdl) {
    std::cout << "Connection established!" << std::endl;
    c->send(hdl, "Hello WebSocket Server", websocketpp::frame::opcode::text);  // 发送消息
}

// 接收到服务器消息时的回调
void on_message(client* c, websocketpp::connection_hdl hdl, client::message_ptr msg) {
    std::cout << "Received from server: " << msg->get_payload() << std::endl;
}

int main() {
    try {
        // 初始化客户端
        ws_client.init_asio();

        // 设置事件回调
        ws_client.set_open_handler(&on_open);
        ws_client.set_message_handler(&on_message);

        // 创建WebSocket连接
        websocketpp::uri uri("ws://localhost:9002");
        websocketpp::connection_hdl hdl = ws_client.get_connection(uri, websocketpp::uri::scheme::ws);

        // 启动连接
        ws_client.connect(hdl);

        std::cout << "Client connected to ws://localhost:9002" << std::endl;

        // 进入事件循环，等待并处理服务器响应
        ws_client.run();
    } catch (websocketpp::exception const& e) {
        std::cerr << "Error: " << e.what() << std::endl;
    }
}
```

### 关键技术点：

- **异步连接**：与服务器端类似，客户端也通过`init_asio()`启用了异步IO，能够高效处理连接和消息。
- **WebSocket URI**：客户端通过指定`ws://localhost:9002` URI来连接服务器，URI是WebSocket协议的标识符，与HTTP协议类似。
  
## 处理WebSocket中的常见问题

### 连接断开与重连

WebSocket协议本身支持连接的关闭，但如果连接异常中断，客户端和服务器应该能够处理这些情况。对于客户端，可以在`on_message`回调中检查连接状态，如果连接断开，则尝试重新连接。

### 异常处理

在生产环境中，WebSocket应用通常会遇到各种异常情况，比如网络断开、服务器崩溃等。为了提高系统的鲁棒性，应该在客户端和服务器中加入适当的错误处理机制。例如，在`on_error`回调函数中记录错误信息，或尝试重新连接。

###  消息压缩

WebSocket协议支持对消息进行压缩传输，特别是在处理大量数据时，压缩可以有效减小网络带宽的占用。可以在WebSocket头中添加`Sec-WebSocket-Extensions`字段来协商是否使用消息压缩。

WebSocket协议作为一种双向全双工协议，提供了低延迟、实时的通信能力，非常适合用于需要高实时性的应用场景。通过C++的`websocketpp`库，我们可以轻松实现一个WebSocket客户端和服务器，并通过回调机制实现消息的双向传递。在实际应用中，WebSocket可以与其他技术如HTTP2、WebRTC等结合，构建更加复杂和高效的通信系统。
