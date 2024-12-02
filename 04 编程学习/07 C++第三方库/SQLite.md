---
title: SQLite数据库介绍
date: 2024-09-30 18:03:59
tags: [SQLite,数据库]
categories: [数据库]
---

## SQLite

SQLite是一个本地化的数据库,不需要客户端服务端什么的配置,主打就是轻量化方便化

他也不是一个独立的进程,而是可以根据应用程序的需求,可以进行静态或者动态的连接

而且他是直接存储在磁盘文件的,提供了简单易用的API接口

需要注意的是 在SQLite中一个数据库对应了一个数据库文件

### 常用接口

```cpp
int sqlite3_threadsafe(); 
// 查看是否启用线程安全 0-未启用 1-启用
```

sqlite有三种安全模式

1. 非线程安全模式
2. 线程安全模式(不同的连接在不同的线程是安全的,或者说进程间是安全的,一个句柄不能用于多线程之间)
3. 串行化模式(可以在不同的线程或者进程之间使用同一个句柄)

```cpp
int sqlite3_open(const char* filename, sqlite3 **ppDb);
int sqlite3_open_v2(const char* filename, sqlite3 **ppDb, int flags, const char* *zVfs);
```

这里是创建或者打开数据库的两种操作

第一个参数是文件名,第二个参数是传入一个句柄指针的地址

第三个参数是flag是可以设置文件的打开方式,提供下面的宏可以选择

* SQLITE_OPEN_READWRITE: 读写方式打开数据库文件
* SQLITE_OPEN_CREATE: 不存在数据库文件则创建
* SQLITE_OPEN_NOMUTEX: 多线程模式,只要不同线程使用不同的连接即可保证线程安全
* SQLITE_OPEN_FULLMUTEX: 串行化模式,即打开所有的锁

当返回值为SQLITE_OK (0)表示数据库成功打开 操作成功

如果是其他值的话就代表着有问题,需要查找对应的宏来看到

```cpp
int sqlite_exec(sqlite3, char* sql, int(*callback)(void*, int, char**, char**), void* arg, char ** err);
```

这是一个执行语句的操作函数

这里面需要传入一个回调函数,对查询到的结果进行处理

返回值含义和上一个是一样的

```cpp
int (*callback) (void*, int, char**, char**);
```

* void* 是用来设置回调时传入的arg参数,一般来说是用于将数据存于某个地址
* int 一行中的列数
* char** 存储一行数据的字符指针数组
* char** 每一列的字段名
* int返回值,成功处理需要返回一个0,如果返回非0程序会直接中断退出

## 使用示例

```cpp
/*
    封装SqliteHelper类
    提供简单的sqlite数据库操作接口

    1. 创建/打开数据库
    2. 针对打开的数据库执行操作
        1. 表操作
        2. 数据操作
    3. 关闭数据库
*/
#include <iostream>
#include <string>
#include <vector>
#include <sqlite3.h>
#include <functional>
#include "../logs/Xulog.h"

class SqliteHelper
{
public:
    typedef int (*SqliteCallback)(void *, int, char **, char **);
    SqliteHelper(const std::string &dbfile)
        : _dbfile(dbfile), _handler(nullptr)
    {
    }
    bool open(int safe_level = SQLITE_OPEN_FULLMUTEX)
    {
        int ret = sqlite3_open_v2(_dbfile.c_str(), &_handler, SQLITE_OPEN_READWRITE | SQLITE_OPEN_CREATE | safe_level, nullptr);
        if (ret != SQLITE_OK)
        {
            ERROR("打开SQLite数据库失败: %s", sqlite3_errmsg(_handler));
            return false;
        }
        return true;
    }
    bool exec(const std::string &sql, SqliteCallback cb, void *arg)
    {
        int ret = sqlite3_exec(_handler, sql.c_str(), cb, arg, nullptr);
        if (ret != SQLITE_OK)
        {
            ERROR("%s--执行语句失败: %s", sql.c_str(), sqlite3_errmsg(_handler));
            return false;
        }
        return true;
    }
    void close()
    {
        sqlite3_close_v2(_handler);
    }

private:
    std::string _dbfile;
    sqlite3 *_handler;
};

```

## 测试

```cpp
#include "Sqlite3.hpp"
#include <cassert>

int select_stu_callback(void *arg, int col_count, char **result, char **fields_name)
{
    std::vector<std::string> *array = (std::vector<std::string> *)arg;
    array->push_back(result[0]);
    return 0;
}
int main()
{
    SqliteHelper helper("./test.db");
    // 创建或打开数据库文件
    assert(helper.open());
    // 创建表
    const char *ct = "CREATE TABLE IF NOT EXISTS stu (sn INTEGER PRIMARY KEY AUTOINCREMENT, name VARCHAR(32), age INT);";
    assert(helper.exec(ct, nullptr, nullptr));
    // 数据操作语句
    // const char *insert_sql = "INSERT INTO stu (sn, name, age) VALUES (1, 'Morty', 16), (2, 'Summer', 18), (3, 'Rick', 58);";
    // assert(helper.exec(insert_sql, nullptr, nullptr));

    // const char *update_sql = "UPDATE stu SET name='S.Morty' WHERE sn=1;";
    // assert(helper.exec(update_sql, nullptr, nullptr));

    // const char *delete_sql = "DELETE FROM stu WHERE sn=2;";
    // assert(helper.exec(delete_sql, nullptr, nullptr));

    const char *select_sql = "SELECT name FROM stu;";
    std::vector<std::string> arr;
    assert(helper.exec(select_sql, &select_stu_callback, &arr));
    for (auto &name : arr)
    {
        std::cout << name << " ";
    }
    std::cout << std::endl;

    // 关闭数据库
    helper.close();
    return 0;
}
```

