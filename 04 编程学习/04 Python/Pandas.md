---
title: Python中Pandas详解之数据结构
date: 2023-12-25 16:36:52
tags:
  - python
categories:
  - python
---

## Pandas 数据分析

### Pandas 简介

Pandas是Python生态下的一个数据分析包，他对于Python数据分析的意义是十分重大的，他与NumPy的不同之处是支持图标和混杂数据运算的，而NumPy是基于数组构建的内容，他的各种图像生成也十分方便，并且支持各种数据存储文件、数据库、甚至Web中读取数据

### Pandas 安装

和NumPy的安装一样使用

```shell
pip install pandas
```

命令安装即可

### Series 类型数据

Series是Pandas的核心数据结构之一，也是理解DataFrame的基础

#### Series的创建

Series的中文翻译是系列，是一种类似于一维数组的结构，是数组和索引构成的

```python
import pandas as pd
pd.Series(data, index = index)
```

在这两个参数中，data是数据源，可以是整数，字符串等，而默认索引就是数据的标签(label)

例如

```python
a = pd.Series([1, 2, 5, 3, 2])
print(a)
```

![屏幕截图 2023-12-25 170012.png](https://s2.loli.net/2023/12/25/WtFK2NIhbMZp8T5.png)

需要注意的是Series的内部是基于NumPy的N维数组构建的，因此内部的数据需要统一

其次Series增加对应的label作为索引，如果没有显示添加索引，Python会自动添加一个0到n-1的索引值，通常都是索引在左，数值在右边

当然，其中的索引也可以被更改为其他的内容，是类似于Python中的字典

Series还提供了一些简单的统计方法，describe()

例如

```python
print(a.describe())
```

![屏幕截图 2023-12-25 171020.png](https://s2.loli.net/2023/12/25/HFpGdt8C14xMk2u.png)

| 参数  | 说明              |
| ----- | ----------------- |
| count | 数据个数          |
| mean  | 均值              |
| std   | 均方差            |
| min   | 最小值            |
| max   | 最大值            |
| 25%   | 前25%的数据分位数 |
| 50%   | 前50%的数据分位数 |
| 75%   | 前75%的数据分位数 |

#### Series的访问

第一种最简单的访问方式就是通过下标存取Series对象内部的元素

当然也可以用label进行访问

需要说明的是，可以按照任意顺序一次访问多个数据

例如

```python
print(a[[1,2,3]])
```

需要说明的是，同时访问多个数值就需要以列表的形式出现

对于两个Series对象还可以通过append()方法进行叠加操作，用来合并对象

但是当叠加对象时，索引就会混乱，因此当叠加时可以采用ignore_index=True，这样就可以重新添加索引

例如

```python
a1.append(a2,ignore_index=True)
```

#### Series 中向量化操作与布尔索引

类似于NumPy，Pandas也支持广播操作，也就是加减乘除一个标量，

同样的Series也支持用布尔表达式提取符合条件的数值，而且Series也可以作为NumPy函数的一个参数进行数据运算

#### Series的切片

对于Series也可以使用切片操作选取处理其中的值，返回值依然是Series的对象

需要注意的时，与数字的切片不同，用label切片不是左闭右开，而是两边都是闭区间

#### Series的缺失值

在数据处理中会遇到缺失值，在Pandas会用NaN来表示

我们可以使用Pandas中的isnull()和notnull()来检测是否含有缺失值

对于isull()方法，他的返回值是一个布尔值Series对象，True表示为缺失值，False表示不是缺失值，notnull正好与之相反

这对海量数据的操作是十分友好的

#### Series的增与删

当我们要删除Series中的数据时，使用drop方法即可，他的参数就是label，也可以是列表，用于删除多项元素

添加数据就可以使用我们之前所说的append方法即可

#### Series的name

除了index和value，还有两个属性，是name和index.name

name可以立即为数值(value)列的名称，index可以理解为一个特殊的索引列，index.name就是索引列(index)的名称，就相当于是列名的作用

默认情况下都被设置为None，我们也可以通过代码进行直接赋值

### DataFrame 数据类型

如果Series是Excel中的一列，那DataFrame就是Excel中的一张表

#### DataFrame的创建

DataFrame实际上可以理解为数据结构中带标签的二维数组，他由若干个Series构成

构建DataFrame的最常用的方法是先构建一个由列表或NumPy数组组成的字典，再将字典作为DataFrame中的参数

例如

```python
df = pd.DataFrame({'name':['summer','morty','rick','jerry']})
print(df)
```

![屏幕截图 2023-12-25 174125.png](https://s2.loli.net/2023/12/25/LXZFc7vk1WIq4Vn.png)

这样我们就可以用若干字典的项来构建一张表了，左边的数字就是行数

例如

```python
df = pd.DataFrame({'name': ['summer', 'morty', 'rick', 'jerry'],
                   'age': [17 , 14, 60,35],
                   'gender': ['female', 'male', 'male', 'male']})
print(df)
```

![屏幕截图 2023-12-25 174600.png](https://s2.loli.net/2023/12/25/r3PvyeXpHKbRIB7.png)

我们也可以通过NumPy的数组生成DataFrame，我们也可以在创建DataFrame时指定列名和行名

例如

```python
df2 = pd.DataFrame(data,columns=['one','two'],index=['a','b'])
```

本质上，DataFrame是由若干个Series构成的，那么Series就是DataFrame的天然数据源，同样的DataFrame也支持NumPy转置操作，也可以使用transpose()方法完成转置

#### DataFrame的访问

访问DataFrame的列很方便，因为DataFrame提供了columns属性，可以通过具体的列名称进行访问

例如

```python
df.columns # 访问列名
df.columns.values[0] # 访问元素
```

DataFrame的一个神奇的地方在于，他可以把列明当作为对象中的一个元素进行获取

例如

```python
df.列名
```

想要获取切片就和NumPy的数组操作时一模一样的，这里不多赘述

#### DataFrame的删除

类似于Series，删除操作也是使用drop方法删除一行或一列

我们也可以使用全局内置函数del，直接在DF中删除某行某列，因为drop方法是生成新的DF

#### DataFrame的添加

##### 添加行

我们直接创建一个空的DF对象，再利用for循环逐个添加新的行即可

例如

```python
df = pd.DataFrame(columns = ['1','2'])
for index in range(5):
    df.loc[index] = ['name '+str(index)] + list(randint(10, size=2))
```

使用loc方法就可以添加一个新的行，index就是DF对象中原先没有的行索引，再进行赋值

我们还可以使用append方法，可以同时批量添加多行数据，类似于NumPy的vstack，垂直堆叠的效果

##### 添加列

和添加行相比就更加简单，实际上就是添加了一个列的名称，再对其内容进行赋值，我们也可以使用concat方法，类似于hstack，水平堆叠，将两个DF对象进行拼接

对于以上两种添加方法，也可以设置忽略原有的索引，进行重新定义索引的编号

对于Pandas的操作指令还有非常多，我们需要在实践中不断掌握，慢慢摸索，越用越熟，越用越精


### Pandas的文件读取与分析

Pandas是数据分析的重要工具，因此从外部读取数据的能力是十分重要的，常用的API如图

| 文件类型   | 文件说明                    | 读取函数        | 写入函数      |
| ------ | ----------------------- | ----------- | --------- |
| CSV    | 是以纯文本形式存储的，以逗号分隔的表格数据   | read_csv    | to_csv    |
| HDF    | 是一种高效存储和分发科学数据的层级数据格式   | read_hdf    | to_hdf    |
| SQL    | 是一种用格式化查询语言编写的数据库查询脚本文件 | read_sql    | to_sql    |
| JSON   | 一种轻量级文本数据交换格式文件         | read_json   | to_json   |
| html   | 一种由超文本标记语言编写的网页文件       | read_html   | read_html |
| PICKLE | Python内部支持的一种序列化文件      | read_pickle | to_pickle |

#### 利用Pandas读取文件

Pandas可以读取到表格类型数据，转换成DF类型的数据，然后通过DF进行数据分析，数据预处理等操作

对于Pandas来说他的核心在于数据分析，而不是进行读写

需要注意的是，读取文件的方法配置了大量的参数，更多内容还需要阅读Pandas的官方文档

#### DataFrame的常用属性

这里一般就是用于显示DF对象中的数据

| 属性    | 说明                            |
| ------- | ------------------------------- |
| dtypes  | 返回各个列的数据类型            |
| columns | 返回各个列的名称                |
| axes    | 返回行标签和列标签              |
| ndim    | 返回维度                        |
| size    | 返回元素个数                    |
| shape   | 返回一个元组，表示几行几列      |
| values  | 返回以个存储DF数值的NumPy的数组 |

#### DataFrame的常用方法

| 方法                | 说明                                   |
| ------------------- | -------------------------------------- |
| head([n])/tail([n]) | 前n行或者后n行数据，方括号表示可选参数 |
| describe()          | 返回所有列的统计信息                   |
| max()/min()         | 返回所有列的最大值和最小值             |
| mean()/median()     | 返回均值和中位数                       |
| std()               | 返回标准差                             |
| sample([n])         | 从DF对象中随机抽取n个样本              |
| dropna()            | 删除所有缺失值的数据                   |
| count()             | 对符合条件的记录计数                   |
| value_counts()      | 查看某列有多少不同值                   |
| groupby()           | 按照给定条件分组                       |

#### DataFrame的条件过滤

和Series一样，我们可以通过布尔索引来提取DF的子集，过滤我们不需要的数据

#### DataFrame的切片操作

他的切片操作和NumPy二维数组的几乎一模一样，而且由于DF具有行标签和列标签，使他的切片操作更加方便了

#### DataFrame的排序操作

在DF中，我们可以根据某列或某几列对DF中的数据进行排序，默认升序

在sort_values方法中参数ascending为升序的意思，默认值为True

### Pandas的聚合与分组运算

对数据分组并进行运算统计，是数据分析的重要环节

#### 聚合

聚合是将多个数值按照某种规则聚合在一起，变成单个数据的转换过程

聚合的流程如下，先根据一个或多个键，拆分我们的Series或者DataFrame，然后根据每个数据块进行统计意义上的各种操作，例如平均值中位数等操作，还可以包含用户自定义的函数

我们可以通过agg方法来实施聚合操作，实际上其中的各个参数，才是其中的精华，通过设置参数，可以将函数作用在一个列或者多个列上

参数的函数名称是有官方提供的，以字符串形式出现，多个参数放在一个列表中即可

例如

```python
df.列名.agg(['min','max','mean','median'])
```

我们就可以通过这种方式获取这一列的最小最大值，平均数与中位数了

需要注意的是，如果是自定义函数，就要直接给出函数名，而不应该传入字符串，同时也不需要括号

#### 分组

groupby()是Pandas的一个高效的分组方法，可以通过各种名称，指标等内容对数据进行分组，再对其进行数据统计与分析

实际上如果我们单纯分组是没有什么意义的，分组的精髓就在于再次使用之前的统计方法，例如mean()、count()等

需要注意的是，我们如果想要获取分组之后的列数据，再将其合并，如果用双方括号，返回值就是DF对象，如果是单括号，返回值就是一个Series对象

例如

```python
df.groupby(分组依据)[[列名]].mean()
df.groupby(分组依据)[列名].mean()
```

第一行就是返回值是一个DF对象，第二行则是Series对象

与之对应的，我们可以在分组之后再进行聚合运算，这样便能很方便的统计出对应数据的大量信息了

例如

```python
df.groupby(分组依据)[列名].agg(['mean','std','skew'])
```

一般来说分组和聚合结合起来使用可以达到很好的效果

DataFrame还有许多其他的工具和特性，例如透视表，类SQL操作等如果想要系统学习还是需要各位通过官方文档，现用现查的方式

### DataFrame中数据清洗方法

对于我们从巨大量级的互联网获取的数据，有很多都包含了不统一，不准确，有缺失的情况，因此我们在进行数据分析之前一定要进行数据清理

| 函数                     | 说明                         |
| ------------------------ | ---------------------------- |
| isnull()                 | 存在缺失值返回True           |
| notnull()                | 不存在缺失值返回True         |
| filna(0)                 | 给缺失值赋值，默认值为0      |
| dropna()                 | 存在缺失值，无条件删除       |
| dropna(how='nall')       | 一行或一列全是缺失值则删除   |
| dropna(axis=1,how='all') | 当列方向全是缺失值则删除列   |
| dropna(axis=1,how='any') | 列只要存在一个缺失值，删除列 |
| dropna(thresh=5)         | 行的有效值低于5个时，删除行  |

