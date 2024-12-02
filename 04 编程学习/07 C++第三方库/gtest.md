---
title: GTest测试框架介绍
date: 2024-10-01 14:37:24
tags:
  - 框架
categories:
  - 框架
---

GTest是谷歌发布的一个跨平台的单元测试框架,主要是为了在不同平台上编写的C++单元测试而生成的

提供了丰富的断言,致命和非致命的判断,参数化

## GTest使用

### 简单的宏断言

断言分两类

一类是ASSERT系列的,如果当前检测失败则直接退出函数

另一类是EXPECT系列,如果当前点检测失败则继续往下执行

但是必须要在单元测试宏函数中才能使用

是这样用的

```cpp
#include <iostream>
#include <gtest/gtest.h>
#include "../logs/Xulog.h"

TEST(test, great_than)
{
    int age = 20;
    ASSERT_GT(age, 18);
    INFO("OK!");
}
int main(int argc, char* argv[])
{
    testing::InitGoogleTest(&argc, argv);
    RUN_ALL_TESTS();
    return 0;
}
```

断言其实不怎么好用 真正有用的其实是事件机制

### 事件机制

GTest中有三种测试事件机制,分别是全局,局部

测试程序中可以有很多测试套件,对应着全局,每一个测试套件中可以有多个单元测试

主要针对整个程序,全局变量

测试中可以有多个测试套件,可以包含一组单元测试,不是很好理解,可以认为这是一个测试环境

可以在单元测试之前进行测试环境初始化,测试完毕后,进行测试环境清理

在整体的测试中,只会初始化一次环境

这其实就是我们用户自己定义的一个测试环境,是一个全局的测试环境类

这其中有一个接口是`virtual void Setup() override();`是用于对测试环境的初始化

还有一个是`virtual void TearDown();`是在代码执行完毕的情况下执行的

在用例测试中,每一次单元测试都是单独进行的,互不影响

全局中是不能定义成员变量的

### 全局使用样例

```cpp
#include <iostream>
#include <gtest/gtest.h>
#include <unordered_map>

#include "../logs/Xulog.h"

class MyEnvironment : public testing::Environment
{
public:
    virtual void SetUp() override
    {
        INFO("单元测试环境初始化!");
    }
    virtual void TearDown() override
    {
        INFO("单元测试结束销毁!");
    }

private:
};

// TEST(MyEnvironment, test1)
// {
//     INFO("单元测试1");
// }

// TEST(MyEnvironment, test2)
// {
//     INFO("单元测试2");
// }

std::unordered_map<std::string, std::string> mp;
class MyMapTest : public testing::Environment
{
public:
    virtual void SetUp() override
    {
        INFO("单元测试环境初始化!");
        mp.insert(std::make_pair("hello","你好"));
    }
    virtual void TearDown() override
    {
        INFO("单元测试结束销毁!");
        mp.clear();
    }
};

TEST(MyMapTest, test1)
{
    INFO("单元测试2");
    ASSERT_EQ(mp.size(),1);
    mp.insert(std::make_pair("aa","AA"));
}

TEST(MyMapTest, test2)
{
    INFO("单元测试2");
    ASSERT_EQ(mp.size(),1);
    mp.erase("aa");
}

int main(int argc, char *argv[])
{
    testing::InitGoogleTest(&argc, argv);
    testing::AddGlobalTestEnvironment(new MyEnvironment);
    testing::AddGlobalTestEnvironment(new MyMapTest);
    RUN_ALL_TESTS();
    return 0;
}
```

### 局部使用样例

```cpp
#include <iostream>
#include <gtest/gtest.h>
#include <unordered_map>

#include "../logs/Xulog.h"

class MyTest : public testing::Test
{
public:
    static void SetUpTestCase()
    {
        INFO("初始化总环境!");
    }
    static void TearDownTestCase()
    {
        INFO("清理总环境");
    }

public:
    std::unordered_map<std::string, std::string> _mp;
};
TEST_F(MyTest, insert_test)
{
    _mp.insert(std::make_pair("hello", "你好"));
    _mp.insert(std::make_pair("aa", "AA"));
}
TEST_F(MyTest, size_test)
{
    ASSERT_EQ(_mp.size(), 2);
}

int main(int argc, char* argv[])
{   
    testing::InitGoogleTest(&argc, argv);
    RUN_ALL_TESTS();
    return 0;
}
```

如果局部需要设置全局的环境,就需要使用之前的全局的继承`void SetUp() override`和`void TearDown() override`
