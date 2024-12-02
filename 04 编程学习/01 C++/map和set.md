---
title: map和set的介绍和使用
date: 2024-03-13 17:08:03
tags: [C++,数据结构,STL]
categories: [C++]
---



## map和set

### 关联式容器

我们在之前讲过STL的一些基础容器，例如vector，list，deque，forward_list等

这些其实统一都称为序列式容器，因为其底层都是线性的序列数据结构，而且存储的内容是元素本身

关联式容器也是存储数据的容器，不同的是，里面存储的是<key,value>的键值对，类似于我们之前讲的二叉搜索树的KV模型

### 键值对

键值对是我们用来表示一一对应关系的结构，包含key和value两个成员变量

key表示关键字，类似于字典中的单词，value表示具体的值，也就是字典中对应的具体释义

在C++中使用pair表示一对值，但并没有具体的对应关系，只是将这一对值进行了封装，可以读取和写入，当然pair也有一些非常有意思的内容，当我们观察其源码时可以看出

这里给出SGI-STL对pair的定义

```c++
template<class T1, class T2>
struct pair
{
	typedef T1 first_type;
	typedef T2 second_type;
   	
    T1 first;
    T2 second;
    
    pair() // 构造
        :first(T1())
        ,second(T2())
    {}
    
    pair(const T1& a, const T2& b) // 拷贝构造
    	:first(a)
    	,second(b)
    {}
};
```

### set

#### 介绍

[set的文档](https://legacy.cplusplus.com/reference/set/set/)

set直接翻译成中文是集合，而他与数学意义上的集合也有不同的地方，set是要求有序的，其次set中的元素不能被修改，元素是const的，但是允许插入删除

set的有序是可以自行定义升序或者降序的，默认是按照小于排序（升序），不允许重复值

set容器通过key访问单个元素的速度比unordered_set慢，但是都允许根据顺序对子集直接迭代

set在底层是使用二叉搜索树的红黑树实现的

set的底层实际上也是有pair的，只不过是<value,value>的形式，在插入元素时，只需要插入value即可

set查找的时间复杂度是$\log_2n$

#### 模板参数

![屏幕截图 2024-03-13 172738.png](https://s2.loli.net/2024/03/13/gNhVZAGJf2c9i8l.png)

第一行的T是存放元素的类型，实际是存在底层的<value,value>键值对

第二行的Compare是仿函数，用于确定是升序还是降序，默认为升序

第三行是一个空间配置器

这里其他的内容参考set的官方文档即可

### map

#### 介绍

[map文档](https://legacy.cplusplus.com/reference/map/map/)

map是关联容器，直接翻译称为映射，是我们之前讲的二叉搜索树的KV模型，他存储的元素是一个有key和value构成的有序对

在map中，key是唯一的，用于标识value

STL中使用typedef来减少长度

```cpp
typedef pair<const key, T> value_type;
```

map内部是对key值排序，默认是升序，他对单个元素的访问速度比unordered_map慢，但是map允许根据顺序对元素进行直接迭代

map支持下标访问，可以直接找到与key对应的value，这里的实现让人赞叹

map的底层是由红黑树实现的，我们在之后的内容里也会使用红黑树实现set和map

#### 模板参数

![image.png](https://s2.loli.net/2024/03/13/84eIfZYUrgpVqBs.png)

Key是键值对中的key，T对应的是value，后面分别是仿函数和空间配置器

需要注意的是，这里的map::key_type和map::mapped_type是typedef出的

![image.png](https://s2.loli.net/2024/03/13/2ngStZE4qOKkTyx.png)

这里我们注意到一个细节，在定义pair的时候，first被const了，说明key值是不能被修改的

#### 为什么map支持下标访问

我们看map对方括号的重定义

![image.png](https://s2.loli.net/2024/03/13/i9wMJVmSO7fZD6x.png)

![image.png](https://s2.loli.net/2024/03/13/Kiep9Ty1vn7kJtx.png)

也就是说他内部是这样实现的

```cpp
mapped_type& operator[] (const key_type& k)
{
    return *((this->insert(make_pair(k,mapped_type())).first)).second;
}
```

这里看上去比较复杂，我们剥洋葱一样的来看

第一层是调用了map的insert插入函数，我们来看insert的定义

![image.png](https://s2.loli.net/2024/03/13/vVWyjzTp37tsnEA.png)

这里调用的是单参数的insert，value_type是一个pair对象，我们可以通过make_pair的名称就知道，他是用于构建pair对象的

insert的返回值是也一个pair对象，first是迭代器，bool是用来表示插入是否成功，如果成果，first指向的就是新pair，如果失败first指向的就是已经存在的pair

然后来看看make_pair函数的定义

![image.png](https://s2.loli.net/2024/03/13/FkUeilswA97CYou.png)

这里就和我们设想的是一样的了，用于构建pair

然后我们看回去，他的参数有两个k和mapped_type()，这里的mapped_type实际上就是value的构造函数，因为在我们插入时只指定了key，没有value，因此直接调用value的构造函数即可

然后构造完成之后，返回的pair被插入到map中，返回值就是pair<iterator, bool>，这时我们取第一个iterator，表示指向插入的pair（无论是否成功插入，都会指向），然后再取第二个值，也就是插入之后的pair的value

返回值也就是pair之中的value

这里有个细节，重定义的返回值是引用的，说明可以支持赋值操作，初始化操作等

例如

```cpp
map<string, string> m;
m["加"] = "plus";
m["减"] = "sub";
m["加"] = "add";
m["乘"];
```



总结一下就是分四步

1. 用<key,value_type()>构建一个键值对，调用inset()插入到map中
2. 如果key存在，插入失败，返回key所在位置的迭代器
3. 如果key不存在，插入成功，返回新插入元素位置的迭代器
4. 返回插入的键值对的引用

### multiset

#### 介绍

[multiset文档](https://legacy.cplusplus.com/reference/set/multiset/)

multiset与set类似，其中的元素是可以重复的，他的底层也是红黑树

红黑树是二叉搜索树，我们可以先按照二叉搜索树理解，他为什么也可以有序呢

实际上是因为他的插入规则是确定的，例如小于在左子树，大于等于在右子树，这样他的中序遍历就依然是有序的

### multimap

[multimap文档](https://legacy.cplusplus.com/reference/map/multimap/)

multimap与map类似，需要注意的是，multimap不支持下标访问了，其他都是相同的

