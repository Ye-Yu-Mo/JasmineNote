---
title: MySQL之数据库DML
date: 2024-01-24 13:49:50
tags: [MySQL,SQL,数据库]
categories: [MySQL]
---

### 数据操作DML

这里的数据操作就是增删改的更新操作，不包括查询

#### 插入

```mysql
insert into 表 (列名1,列名2,列名3...) values (值1,值2,值3...); //向表中插入对应列
insert into 表 values (值1,值2,值3...);     //向表中插入所有列
```

第一种是需要按照列名对应写数值的，可以省略，但必须对应

第二种是一次插入一行，是都要写的

也可以插入多行只需在之后填入新的行即可

```mysql
insert into student(sid,name,gender,age,birth,address,score)
            values(1001,'男',18,'1996-12-23','北京',83.5),
            	  (1001,'男',18,'1996-12-23','北京',83.5);
insert into student values(1001,'男',18,'1996-12-23','北京',83.5);
```

#### 修改

```mysql
update 表名 set 字段名=值,字段名=值...;
update 表名 set 字段名=值,字段名=值... where 条件;
```

例如

```mysql
-- 将所有学生的地址修改为重庆 
update student set address = '重庆';

-- 讲id为1004的学生的地址修改为北京 
update student set address = '北京' where id = 1004 

-- 讲id为1005的学生的地址修改为北京，成绩修成绩修改为100 
update student set address = '广州',score=100 where id = 1005
```

#### 删除

```mysql
delete from 表名 [where 条件];
truncate table  表名;
truncate 表名;
```

例如

```mysql
-- 1.删除sid为1004的学生数据
delete from student where sid  = 1004;

-- 2.删除表所有数据
delete from student;

-- 3.清空表数据
truncate table student;
truncate student;
```

需要注意的是delete和truncate原理不同，delete只删除内容，而truncate类似于drop table ，可以理解为是将整个表删除，然后再创建该表。之后再学习其他内容的时候我们会继续补充其中的内容。
