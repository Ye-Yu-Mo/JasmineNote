---
title: STL与string的介绍
date: 2023-12-06 15:30:25
tags:
  - C++
  - STL
categories:
  - C++
  - STL
---

# STL介绍

## 什么是STL

STL的全拼是standard template libaray（标准模板库），这是C++标准库中的重要组成，是一个数据结构与算法的基本框架

STL其实可以看作一个标准，他对各种数据结构、算法等内容做了功能上的限定，因此不同的人在实现相同的部分可能会有不同的实现方式，也就产生了不同的STL版本

SGI版本的STL的可阅读性非常高，在学习过程中主要参考这个版本

# string介绍

## string类的介绍

string类是C++中的一个容器，对应的是C语言的字符串，而C语言对字符串的处理是使用字符数组的方法，相对来说比较繁琐和相对不安全，由此在C++引出了string类

## STL中的string类

### string的构造函数

| 函数名称                 | 功能                   |
| ------------------------ | ---------------------- |
| string()                 | 构造空的string对象     |
| string(const char* s)    | 用字符串构造string对象 |
| string(size_t n, char c) | n个字符c构造string对象 |
| string(const string& s)  | 拷贝构造               |

### string的容量操作

| 函数名称   | 功能           |
| ---------- | -------------- |
| size()     | 字符串有效长度 |
| length()   | 字符串有效长度 |
| capacity() | 空间大小       |
| empty()    | 检测是否为空   |
| clear()    | 清空           |
| reserve()  | 预留空间       |
| resize()   | 保留字符串     |

### string的访问

| 函数名称     | 功能                                                         |
| ------------ | ------------------------------------------------------------ |
| operator[]   | 返回pos位置的字符                                            |
| begin、end   | begin获取第一个字符的迭代器、end获取最后一个字符下一个位置的迭代器 |
| rbegin、rend | 与begin、end相反                                             |
| 范围for      | 利用迭代器进行遍历                                           |

### string的增删查改

| 函数名称   | 功能                  |
| ---------- | --------------------- |
| push_back  | 尾插                  |
| append     | 追加                  |
| operator+= | 追加                  |
| c_str      | 转换为C语言中的字符串 |
| find       | 从pos位置往后找       |
| rfind      | 从pos位置往前找       |
| substr     | 从pos位置开始截取     |

### string的非成员函数

| 函数名称             | 功能                 |
| -------------------- | -------------------- |
| operator+            | 加和，传值返回效率低 |
| operator>>           | 输入                 |
| operator<<           | 输出                 |
| getline              | 获取一行字符         |
| relational operators | 比较大小             |
