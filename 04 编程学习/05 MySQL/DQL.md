---
title: MySQL之数据库DQL
date: 2024-02-01 11:23:45
tags: [MySQL,SQL,数据库]
categories: [MySQL]
---

## 数据查询DQL

数据库管理系统一个重要功能就是数据查询，数据查询不应只是简单返回数据库中存储的数据，还应该根据需要对数据进行筛选以及确定数据以什么样的格式显示。

MySQL提供了功能强大、灵活的语句来实现这些操作。

MySQL数据库使用select语句来查询数据。

### 基本查询

语法格式

```sql
select 
	[all|distinct]
	<目标列的表达式1> [别名],
	<目标列的表达式2> [别名]...
from <表名或视图名> [别名],<表名或视图名> [别名]...
[where<条件表达式>]
[group by <列名> 
[having <条件表达式>]]
[order by <列名> [asc|desc]]
[limit <数字或者列表>];
```

这里的关键字太多了

where就是第一步的筛选条件，group by是分组条件，having是第二步的筛选条件，order by是排序，limit是限制条数

我们用到的最多的就是三个关键字

```sql
select *| 列名 from 表 where 条件
```

例如

```sql
-- 创建数据库
create database if not exist mydb2;
use mydb2;
-- 创建商品表：
create table product(
    pid int primary key auto_increment, -- 商品编号
    pname varchar(20) not null , -- 商品名字
    price double,  -- 商品价格
    category_id varchar(20) -- 商品所属分类
);

-- 添加数据
insert into product values(null,'海尔洗衣机',5000,'c001');
insert into product values(null,'美的冰箱',3000,'c001');
insert into product values(null,'格力空调',5000,'c001');
insert into product values(null,'九阳电饭煲',200,'c001');

insert into product values(null,'啄木鸟衬衣',300,'c002');
insert into product values(null,'恒源祥西裤',800,'c002');
insert into product values(null,'花花公子夹克',440,'c002');
insert into product values(null,'劲霸休闲裤',266,'c002');
insert into product values(null,'海澜之家卫衣',180,'c002');
insert into product values(null,'杰克琼斯运动裤',430,'c002');
 
insert into product values(null,'兰蔻面霜',300,'c003');
insert into product values(null,'雅诗兰黛精华水',200,'c003');
insert into product values(null,'香奈儿香水',350,'c003');
insert into product values(null,'SK-II神仙水',350,'c003');
insert into product values(null,'资生堂粉底液',180,'c003');
 
insert into product values(null,'老北京方便面',56,'c004');
insert into product values(null,'良品铺子海带丝',17,'c004');
insert into product values(null,'三只松鼠坚果',88,null);
      
-- 简单查询
-- 1.查询所有的商品.  
select * from product;
-- 2.查询商品名和商品价格. 
select pname,price from product;
-- 3.别名查询.使用的关键字是as（as可以省略的）.  
-- 3.1表别名: 
select * from product as p;
-- 3.2列别名：
select pname as pn from product; 
-- 4.去掉重复值.  
select distinct price from product;
-- 5.查询结果是表达式（运算查询）：将所有商品的价格+10元进行显示.
select pname,price+10 from product;
```

#### 运算符

这里的运算符表示的就是对表中数据进行的运算，分为算术运算符，比较运算符，逻辑运算符，位运算符

##### 算数运算符

| **算术运算符**       | **说明**           |
| -------------------- | ------------------ |
| **+**                | 加法运算           |
| **-**                | 减法运算           |
| *****                | 乘法运算           |
| **/** **或** **DIV** | 除法运算，返回商   |
| **%** **或** **MOD** | 求余运算，返回余数 |

##### 比较运算符

| **比较运算符**                | **说明**                                                     |
| ----------------------------- | ------------------------------------------------------------ |
| **=**                         | 等于                                                         |
| **<**  **和**  **<=**         | 小于和小于等于                                               |
| **>**  **和**  **>=**         | 大于和大于等于                                               |
| **<=>**                       | 安全的等于，两个操作码均为NULL时，其所得值为1；而当一个操作码为NULL时，其所得值为0 |
| **<> 或!=**                   | 不等于                                                       |
| **IS NULL** **或** **ISNULL** | 判断一个值是否为  NULL                                       |
| **IS NOT NULL**               | 判断一个值是否不为  NULL                                     |
| **LEAST**                     | 当有两个或多个参数时，返回最小值                             |
| **GREATEST**                  | 当有两个或多个参数时，返回最大值                             |
| **BETWEEN AND**               | 判断一个值是否落在两个值之间                                 |
| **IN**                        | 判断一个值是IN列表中的任意一个值                             |
| **NOT IN**                    | 判断一个值不是IN列表中的任意一个值                           |
| **LIKE**                      | 通配符匹配                                                   |
| **REGEXP**                    | 正则表达式匹配                                               |

##### 逻辑运算符

| **逻辑运算符**           | **说明** |
| ------------------------ | -------- |
| **NOT** **或者** **!**   | 逻辑非   |
| **AND** **或者** **&&**  | 逻辑与   |
| **OR** **或者** **\|\|** | 逻辑或   |
| **XOR**                  | 逻辑异或 |

##### 位运算符

| **位运算符** | **说明**               |
| ------------ | ---------------------- |
| **\|**       | 按位或                 |
| **&**        | 按位与                 |
| **^**        | 按位异或               |
| **<<**       | 按位左移               |
| **>>**       | 按位右移               |
| **~**        | 按位取反，反转所有比特 |

需要注意的是，位运算符是按照二进制进行运算，其运算的结果会转化为十进制数

例如

```sql
-- 查询商品名称为“海尔洗衣机”的商品所有信息：
select * from product where pname = '海尔洗衣机';
 
-- 查询价格为800商品
select * from product where price = 800;
 
-- 查询价格不是800的所有商品
select * from product where price != 800;
select * from product where price <> 800;
select * from product where not(price = 800);
 
-- 查询商品价格大于60元的所有商品信息
select * from product where price > 60;
 
 
-- 查询商品价格在200到1000之间所有商品
select * from product where price >= 200 and price <=1000;
select * from product where price between 200 and 1000;

-- 查询商品价格是200或800的所有商品
select * from product where price = 200 or price = 800;
select * from product where price in (200,800);
 
-- 查询含有‘裤'字的所有商品
select * from product where pname like '%裤%';
-- 这里的%就表示任意内容，只要包含裤字就可以
 
-- 查询以'海'开头的所有商品
select * from product where pname like '海%';
 -- 这里就表示以海开头，后面是任意内容，用%代替
 
-- 查询第二个字为'蔻'的所有商品
select * from product where pname like '_蔻%';
-- 这里的下划线_就表示一个任意字符
 
-- 查询category_id为null的商品
select * from product where category_id is null;
 
-- 查询category_id不为null分类的商品
select * from product where category_id is not null;

-- 使用least求最小值
select least(10, 20, 30); -- 10
select least(10, null , 30); -- null
 
-- 使用greatest求最大值
select greatest(10, 20, 30);
select greatest(10, null, 30); -- null

```

#### 排序查询

如果需要对读取的数据进行排序，就可以使用 MySQL 的 order by 子句来设定你想按哪个字段哪种方式来进行排序，再返回搜索结果。

```sql
select 
 字段名1，字段名2，……
from 表名
order by 字段名1 [asc|desc]，字段名2[asc|desc]……
```

> 1. asc代表升序，desc代表降序，如果不写默认升序
>
> 2. order by用于子句中可以支持单个字段，多个字段，表达式，函数，别名
>
> 3. order by子句，放在查询语句的最后面。LIMIT子句除外



例如

```sql
-- 1.使用价格排序(降序)
select * from product order by price desc;
-- 2.在价格排序(降序)的基础上，以分类排序(降序)
select * from product order by price desc,category_id asc;
-- 3.显示商品的价格(去重复)，并排序(降序)
select distinct price from product order by price desc;
```

#### 聚合查询

之前我们做的查询都是横向查询，它们都是根据条件一行一行的进行判断，而使用聚合函数查询是纵向查询，它是对一列的值进行计算，然后返回一个单一的值；另外聚合函数会忽略空值。

| **聚合函数** | **作用**                                                     |
| ------------ | ------------------------------------------------------------ |
| **count()**  | 统计指定列不为NULL的记录行数；                               |
| **sum()**    | 计算指定列的数值和，如果指定列类型不是数值类型，那么计算结果为0 |
| **max()**    | 计算指定列的最大值，如果指定列是字符串类型，那么使用字符串排序运算； |
| **min()**    | 计算指定列的最小值，如果指定列是字符串类型，那么使用字符串排序运算； |
| **avg()**    | 计算指定列的平均值，如果指定列类型不是数值类型，那么计算结果为0 |

例如

```sql
-- 1 查询商品的总条数
select count(*) from product;
-- 2 查询价格大于200商品的总条数
select count(*) from product where price > 200;
-- 3 查询分类为'c001'的所有商品的总和
select sum(price) from product where category_id = 'c001';
-- 4 查询商品的最大价格
select max(price) from product;
-- 5 查询商品的最小价格
select min(price) from product;
-- 6 查询分类为'c002'所有商品的平均价格
select avg(price) from product where category_id = 'c002';
```

> 1. count函数对null值的处理
>
>    如果count函数的参数为星号（*），则统计所有记录的个数。而如果参数为某字段，不统计含null值的记录个数。
>
> 2. sum和avg函数对null值的处理
>
>    这两个函数忽略null值的存在，就好象该条记录不存在一样。
>
> 3. max和min函数对null值的处理
>
>     max和min两个函数同样忽略null值的存在。

例如

```sql
-- 创建表
create table test_null( 
 c1 varchar(20), 
 c2 int 
);

-- 插入数据
insert into test_null values('aaa',3);
insert into test_null values('bbb',3);
insert into test_null values('ccc',null);
insert into test_null values('ddd',6);
 
-- 测试
select count(*), count(1), count(c2) from test_null;
select sum(c2),max(c2),min(c2),avg(c2) from test_null;
```

#### 分组查询

使用group by语句对查询信息分组

语法

```sql
select 字段1,字段2… from 表名 group by 分组字段 having 分组条件;
```

例如

```sql
-- 1 统计各个分类商品的个数
select category_id ,count(*) from product group by category_id \;
```

如果要进行分组的话，则SELECT子句之后，只能出现分组的字段和统计函数，其他的字段不能出现

需要注意的是分组查询之后的筛选条件必须使用having，而不能使用where

> * 分组之后对统计结果进行筛选的话必须使用having，不能使用where
> * where子句用来筛选 FROM 子句中指定的操作所产生的行
> * group by 子句用来分组 WHERE 子句的输出。
> * having 子句用来从分组的结果中筛选行

语法

```sql
select 字段1,字段2… from 表名 group by 分组字段 having 分组条件;
```

例如

```sql
-- 2.统计各个分类商品的个数,且只显示个数大于4的信息
select category_id ,count(*) from product group by category_id having count(*) > 1;
```

#### 分页查询

分页查询在项目开发中常见，由于数据量很大，显示屏长度有限，因此对数据需要采取分页显示方式。例如数据共有30条，每页显示5条，第一页显示1-5条，第二页显示6-10条。 

语法

```sql
-- 方式1-显示前n条
select 字段1，字段2... from 表明 limit n
-- 方式2-分页显示
select 字段1，字段2... from 表明 limit m,n
-- m: 整数，表示从第几条索引开始，计算方式 （当前页-1）*每页显示条数
-- n: 整数，表示查询多少条数据
```

例如

```sql
-- 查询product表的前5条记录 
select * from product limit 5 

-- 从第4条开始显示，显示5条 
select * from product limit 3,5
```

#### INSERT INTO SELECT语句

将一张表的数据导入到另一张表中，可以使用INSERT INTO SELECT语句 。

语法

```sql
insert into Table2(field1,field2,…) select value1,value2,… from Table1 

insert into Table2 select * from Table1
```

要求table2必须存在

#### SELECT INTO FROM语句

将一张表的数据导入到另一张表中，有两种选择 SELECT INTO 和 INSERT INTO SELECT 。

语法

```sql
SELECT vale1, value2 into Table2 from Table1
```

 要求目标表Table2不存在，因为在插入时会自动创建表Table2，并将Table1中指定字段数据复制到Table2中。
