---
title: 常见开源组件的详解
date: 2024-10-13 18:49:19
tags: [C++]
categories: [C++]
---

## RPC

RPC是一种通信协议 他可以让程序在不同的计算机上调用彼此的程序或者服务 就像本地调用函数一样调用远程服务器的服务

RPC框架负责底层的网络通信 序列化和反序列化数据 错误处理

这样我们在开发的过程中就不再需要重复造轮子了 这和reactor一样只是一种编程思想

各家公司一般都有自己的RPC框架

### RPC架构和工作流程

RPC的工作流程很简单

1. 客户端调用：客户端调用本地的一个代理函数（stub），负责将请求参数序列化，然后发送到服务器

   > 代理函数（stub）：是负责客户端和服务器之间的通信 主要是为远程服务调用提供的一个本地接口 把远程调用的过程隐藏起来 他的内部处理功能就是序列化与反序列化 错误处理 返回结果 这种代理函数也支持多种编程语言 让不同语言的服务也能够通信

2. 网络传输

3. 服务处理：服务端接收代理函数的请求 反序列化参数 调用相关服务

4. 响应返回

5. 客户端接收

![null.png](https://s2.loli.net/2024/10/13/tszDK52SedZJEoh.png)

我们可以在很多地方都看到这种思想 例如MySQL的客户端和服务端分离 RabbitMQ的客户端服务端分离

客户端负责的起始就只有把请求按要求进行序列化和传递请求给服务端 真正运行复杂服务的是服务端

这里就有了两种调用方式 同步调用和异步调用

大家基础好的话看到这两个词就能明白是什么意思 同步调用就是阻塞等待返回结果 而异步调用不等待结果直接继续执行

### 为什么有了HTTP还要用RPC

这两者的共同点都是完成了一个请求与响应的过程

HTTP的协议很简单易用 用途也很广泛 但是RPC更加适用于这种远程调用的过程

#### 底层协议

RPC的底层协议是TCP/IP协议 而HTTP协议是TCP协议

RPC下面可以使用TCP和UDP 也可以使用其他的自定义协议

但是HTTP是只能使用TCP了 在一些高性能低延迟条件下还是RPC更胜一筹 不用每次都三次握手四次挥手

#### 数据格式

RPC主要是用二进制格式数据（例如protobuf） 让数据的传输更加安全高效

| 特性           | Protobuf               | JSON                     | XML                     |
| -------------- | ---------------------- | ------------------------ | ----------------------- |
| **数据体积**   | 小，二进制格式         | 大，文本格式             | 更大，带有大量标记      |
| **解析速度**   | 快，二进制格式         | 较慢，基于字符串解析     | 更慢，带有复杂的结构    |
| **可读性**     | 不可读                 | 人类可读                 | 人类可读                |
| **向后兼容性** | 支持，未识别字段被忽略 | 支持，但需要注意字段命名 | 支持，但 XML 复杂性较高 |
| **多语言支持** | 强，支持多种编程语言   | 较强，流行语言都有支持   | 较强，但通常处理较复杂  |

而HTTP主要是用文本格式的数据 JSON或者XML进行数据交换 虽然也支持二进制传输 但是会导致更大的数据体积

> 因为传统的HTTP通信时是要求以文本形式进行传输的 当传输二进制数据时就要求其转换为一种文本表示 常见的就是Base64编码
> 结构化的JSON和XML的文本格式开销也很大 对于大型的结构化数据也不是很友好

#### 连接管理

当请求很多时 长连接的效率就很高 可以同时发送很多请求

而传统的HTTP则使用短连接 会频繁的建立和关闭连接

#### 错误处理

学过HTTP协议的错误处理都知道 这个玩意的错误机制是不太好的 客户端要根据状态码的不同处理不同的错误情况

### 使用场景

1. 微服务架构 在微服务架构中 各个服务之间需要频繁互动
2. 高性能应用 快速响应和低延迟应用
3. 分布式系统 多个不同地域的服务器进行数据交互与服务调用

### 常见的RPC框架

1. gRPC：google开发的 支持多种语言 基于HTTP/2协议
2. Apache Thrift 支持多种语言的RPC框架 灵活性高 适合构建跨语言的分布式服务
3. Apache Dubbo 高性能的Java RPC框架 提供服务治理功能 适用于微服务架构
4. Hessian 轻量级的二进制RPC框架 适用于Java和其他编程语言的通信

## Web应用框架

Web应用框架是用于开发Web工具集 简化开发的流程 可以快速的构建起web应用

如果有学过Python的同学可以尝试上手一下flask或者django还挺好玩的

### 主要功能

- **请求路由**：将用户请求的 URL 与相应的控制器或方法映射起来。

  > 控制器指的是用于接收用户请求并对其进行处理的部分 控制用户的业务流程

- **中间件支持**：在请求到达控制器之前和返回响应之前执行特定操作。

- **模板引擎**：用于生成动态 HTML 页面或返回 JSON 数据。

- **数据库交互**：提供与数据库的集成方式，简化数据的 CRUD（创建、读取、更新、删除）操作。

- **身份验证和授权**：简化用户登录、会话管理和权限控制。

- **安全特性**：帮助抵御常见的 Web 安全攻击，例如 SQL 注入和跨站脚本攻击（XSS）。

### 常见的Web应用框架

#### Spring Boot (Java)

Spring Boot 是 Java 生态中最流行的 Web 应用框架之一，它基于 Spring 框架，简化了 Spring 应用的配置和部署。Spring Boot 通过“开箱即用”的默认配置，使得开发者可以更快地搭建 Web 应用，而不必关心大量的 XML 或 Java 配置

**主要特点**：

- **自动配置**：Spring Boot 可以根据项目的依赖自动配置常用的组件，如数据库、消息队列等。
- **嵌入式服务器**：支持嵌入式的 Tomcat、Jetty 或 Undertow，简化应用的打包和部署。
- **广泛的生态支持**：与 Spring 家族的其他组件（如 Spring Security、Spring Data）无缝集成。

**适用场景**：

- 适合需要构建复杂的企业级应用和微服务架构的项目。

#### Django (Python)

Django 是一个 Python 框架，以“快速开发”和“代码重用”而著称。它遵循“Django 管理后台”的理念，提供了一个完备的后台管理界面，开发者可以快速上手并生成高效的 Web 应用。

**主要特点**：

- **全栈框架**：内置 ORM（对象关系映射）、身份认证、模板引擎等功能，几乎涵盖了 Web 开发的各个方面。
- **安全性**：Django 内置了很多安全特性，帮助开发者防范常见的 Web 攻击。
- **高度抽象化**：通过简化数据库操作和 URL 路由，使开发者能够以极少的代码实现复杂功能。

适用场景：

- 快速原型设计和中小型 Web 应用，特别是涉及数据库驱动的项目。

#### Express.js (Node.js)

Express.js 是一个极简的 Node.js Web 应用框架。它非常轻量，提供了路由和中间件支持，使得开发者可以根据需求灵活构建 Web 应用或 API 服务。

**主要特点**：

- **轻量化和灵活性**：没有过多的封装，开发者可以自由选择各种库和中间件。
- **丰富的中间件生态**：通过 Express 中间件，可以轻松扩展功能，如日志记录、身份验证、错误处理等。
- **异步编程模型**：基于 Node.js 的非阻塞 I/O 模型，适合高并发场景。

**适用场景**：

- 单页应用（SPA）、RESTful API 和实时应用（如 WebSocket 应用）。

CppCMS

CppCMS 是一个开源的高性能 Web 框架，专为高负载的 Web 应用程序设计。它的特点是通过低级别的 C++ 优化，提供了类似于 Python 的 Django、Ruby on Rails 的框架特性，但性能更高。

**主要特点**：

- **高性能**：专为处理高并发、高流量而设计，擅长于处理静态内容、动态页面生成和缓存管理。
- **模板引擎**：提供了自己的模板引擎，支持动态 HTML 内容的生成。
- **会话管理和认证**：内置会话管理机制，支持用户认证和授权。
- **I18N 支持**：支持国际化（I18N），方便多语言 Web 应用的开发。

**适用场景**：

- 高并发、大流量的 Web 服务。
- 需要细粒度性能优化的 Web 应用。

## Redis

Redis是一个基于内存的NoSQL数据库 通常用于缓存和消息队列 在现代分布式系统和高并发应用中不可或缺 性能高 数据结构丰富 使用灵活

### 主要特点

- **键值对存储**：Redis 以键值对的方式存储数据，所有的数据都存储在内存中。

- **高性能**：由于在内存中进行数据存取，读写速度极快。

- **丰富的数据结构**：Redis 支持多种复杂的数据结构，包括字符串、哈希、列表、集合和有序集合等。

- **持久化**：Redis 支持通过快照（Snapshot）和 AOF（Append Only File）进行数据持久化。

  > 快照：将内存中的数据定期保存到硬盘中 生成一个RDB文件
  >
  > AOF：对Redis数据的写操作以日志的方式追加到文件中 当重启时可以通过操作日志来恢复数据 有点像MySQL的数据备份

- **分布式集群**：Redis 支持分片和复制，提供高可用性和扩展性。

### 应用场景

* 缓存：这也是最常见的应用场景之一，显著加快数据的读取速度 减少数据库的负担
* 存储会话：分布式系统中存储多个服务器的会话 确保多个应用服务器能够访问同一份会话数据
* 队列存储：通过列表、发布订阅功能 做一个轻量级的消息队列系统 适用于任务分发和和异步处理
* 分布式锁：Redis的单线程特性和高性能 可以用于分布式系统的锁管理

### 缓存问题

1. 缓存击穿

   > 当高频访问的热点数据突然失效时 大量请求会涌入数据库 可以通过互斥锁或者请求合并的方式 确保只有一个请求访问数据库 其他请求等待缓存重新设置

2. 内存穿透

   > 当恶意请求频繁访问查询不存在的键时 会直接访问数据库 可以使用布隆过滤器来预先判断这个请求是否应该访问数据库 减少无效查询

3. 缓存雪崩

   > 大量数据同时失效时 会有大量请求涌入数据库 对系统造成巨大压力 主要有下面的方法
   >
   > 1. 设置不同的过期时间 确保缓存不会同时失效
   > 2. 增加随机事件
   > 3. 双缓存 使用本地缓存和Redis缓存的双层架构 减少Redis失效的压力

### Redis集群架构

这是一种将多个Redis组合在一起实现高可用性和高性能的分布式架构

将数据分片存储在多个主节点上 实现负载均衡 故障转移 自动扩展

这里有两种集群架构

#### 主从复制

每个主节点可以有一个或者多个从节点，从节点用于备份主节点的数据，可以在主节点故障时自动升级为新的主节点

#### Redis Cluster

这个架构可以让数据通过哈希槽，分到不同的主节点上 保证集群的容错能力

可以进行自动分片 负载均衡 并且在主节点挂了的适合进行故障转移

> 哈希槽（Hash Slot）是一种机制 Redis将所有的键（用键来算编号）映射到固定数量的哈希槽上 每主节点负责一定数量的哈希槽 这样来实现负载均衡
