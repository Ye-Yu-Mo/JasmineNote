---
title: 现代C++HTTP框架cinatra
date: 2024-11-16 10:53:17
tags: [C++]
categories: [C++]
---

## cinatra简介

cinatra是一个基于C++20协程的高性能HTTP框架，它的目标是提供一个快速开发的C++ HTTP框架解决方案

它不仅支持HTTP/1.1和1.0，还支持SSL和WebSocket，使得开发者可以轻松构建数据库访问服务器、文件上传下载服务器、实时消息推送服务器，甚至是MQTT服务器

### 主要特点

1. **统一而简单的接口**：cinatra提供了一个简洁的API，使得开发者可以快速上手
2. **Header-only**：作为一个头文件库，cinatra无需复杂的编译和链接过程，可以直接包含头文件即可使用
3. **跨平台支持**：cinatra可以在多种操作系统上编译和运行，包括Ubuntu、macOS和Windows
4. **高性能**：cinatra在性能测试中表现出色，是世界上性能最好的HTTP服务器之一。
5. **支持面向切面编程**：cinatra支持AOP（面向切面编程），允许开发者以非侵入式的方式添加日志、安全检查等功能

## 快速上手

### 编译器版本要求

要使用cinatra，你需要一个支持C++20的编译器，如gcc 10.2、clang 13或Visual Studio 2022。

### 使用指南

cinatra是header-only的，你只需引用其include目录，并设置相应的编译选项即可。例如，在Linux下，你可以设置：

```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread -std=c++20")
```

如果使用g++编译，还需要添加：

```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fcoroutines")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -fno-tree-slp-vectorize")
```

### 快速示例

cinatra提供了丰富的示例代码，从简单的“Hello World”到复杂的RESTful服务和WebSocket通信。以下是创建一个简单HTTP服务器的示例代码：

```cpp
#include "cinatra.hpp"
using namespace cinatra;

int main() {
    int max_thread_num = std::thread::hardware_concurrency();
    coro_http_server server(max_thread_num, 8080);
    server.set_http_handler<GET, POST>("/", [](coro_http_request& req, coro_http_response& res) {
        res.set_status_and_content(status_type::ok, "hello world");
    });

    server.sync_start();
    return 0;
}
```

## 项目地址

[cinatra](https://github.com/qicosmos/cinatra)

