---
title: ProtoBuf序列化框架介绍
date: 2024-09-21 20:06:00
tags:
  - 框架
categories:
  - 框架
---

## ProtoBuf介绍

ProtoBuf全称是Protocol Buffer，是一个数据结构的序列化和反序列化框架

他又很多好处，首先是他支持跨平台，支持Java、C++、Python等多种语言，还比XML更小更快更简单

除此之外还可以更新数据结构，不会破坏原有的结构

### 使用流程

<img src="https://s2.loli.net/2024/09/21/F8OHl6pecSR3awh.png" alt="image.png" style="zoom:50%;" />

这个框架的使用流程是这样的

我们需要编写的是.proto文件，来描述结构化的对象，和其中的成员，属性

这个文件可以使用protoc编译器来处理，处理的结果就是我们需要的对应的语言，用来结构化对象数据操作的代码

我们在业务代码中包含这些头文件，就能使用这些方法把我们要序列化或者反序列化的数据进行处理

## QUICK START

这里我们通过一个简单的通讯录实现来快速上手protobuf

### 创建.proto文件

推荐的规范

* 创建该文件，文件名必须用全小写命名，字母之间使用`_`连接
* 2字符缩进

### 注释

与C/C++一样

```protobuf
//
/* */
```

### 语法

```protobuf
syntax = "proto3"; // 指定使用proto3版本语法

package contacts; // 可选的声明符，表示文件的命名空间

// 结构化对象
message contact{
    // 字段描述 ： 字段类型 字段名 = 字段唯一编号;
    uint64 ID = 1;
    string Name = 2;
    string TelNumber = 3;
    string Adders = 4;
}
```

### 编译

生成cpp文件

```shell
protoc --cpp_out=. contacts.proto
```

`.`表示在当前目录生成

### 部分代码展示

```cpp
class contact PROTOBUF_FINAL :
    public ::PROTOBUF_NAMESPACE_ID::Message  {
        
  void CopyFrom(const ::PROTOBUF_NAMESPACE_ID::Message& from) final;
  void MergeFrom(const ::PROTOBUF_NAMESPACE_ID::Message& from) final;

  friend void swap(contact& a, contact& b) {
    a.Swap(&b);
  }


  // string Name = 2;
  void clear_name();
  const std::string& name() const;
  void set_name(const std::string& value);
  void set_name(std::string&& value);
  void set_name(const char* value);
  void set_name(const char* value, size_t size);
        
  // string TelNumber = 3;
  void clear_telnumber();
  const std::string& telnumber() const;
  void set_telnumber(const std::string& value);
  void set_telnumber(std::string&& value);
  void set_telnumber(const char* value);
  void set_telnumber(const char* value, size_t size);

  // string Adders = 4;
  void clear_adders();
  const std::string& adders() const;
  void set_adders(const std::string& value);
  void set_adders(std::string&& value);
  void set_adders(const char* value);
  void set_adders(const char* value, size_t size);

  // uint64 ID = 1;
  void clear_id();
  void set_id(::PROTOBUF_NAMESPACE_ID::uint64 value);
};

class PROTOBUF_EXPORT MessageLite {
 public:
  // 部分反序列化接口
  bool ParseFromString(const std::string& data);
  bool ParsePartialFromString(const std::string& data);
  bool ParseFromArray(const void* data, int size);
  bool ParsePartialFromArray(const void* data, int size);
  // 部分序列化接口
  std::string SerializeAsString() const;
  std::string SerializePartialAsString() const;
  bool SerializeToFileDescriptor(int file_descriptor) const;
  bool SerializePartialToFileDescriptor(int file_descriptor) const;
};

```

这里提供了一些访问和设置的接口，除此之外，这个类继承的是Message类，基类是提供了序列化的反序列化的接口的，我们使用这个框架的目的就正在于此

get的方法名称就是小写的字段名称，set的方法名称是以`set_`开头，后面续上小写字段名

### 使用接口

需要注意的是，protobuf序列化的结果是二进制数据，并非字符，因此直接打印或者读取文件也依然是无法读取的，这里我们只是做序列化的演示

```cpp
#include "contacts.pb.h"
#include <string>

int main()
{
    // 创建对象
    contacts::contact cont;
    // 初始化值
    cont.set_id(0001);
    cont.set_adders("翻斗大街翻斗花园2号楼1001室");
    cont.set_name("胡图图");
    cont.set_telnumber("13600000001");
    // 序列化
    std::string str = cont.SerializeAsString();
    // 数据传输
    // ......

    // 反序列化
    contacts::contact recv;
    bool res = recv.ParseFromString(str);
    if (res == false)
    {
        std::cout << "反序列化失败" << std::endl;
        return -1;
    }
    // 使用
    std::cout << "ID: " << recv.id() << std::endl;
    std::cout << "Name: " << recv.name() << std::endl;
    std::cout << "Adders: " << recv.adders() << std::endl;
    std::cout << "TelNumber: " << recv.telnumber() << std::endl;
    return 0;
}
```

### 运行结果

![image.png](https://s2.loli.net/2024/09/21/ZPAnsimIcDb9Y7o.png)

