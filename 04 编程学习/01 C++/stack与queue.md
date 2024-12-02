---
title: STL中stack与queue详解
date: 2023-12-21 17:38:29
tags: [C++ ,STL,stack,queue]
categories: [C++ ,STL]
---

# stack与queue的基本介绍

在学习和实现完vector，list等容器之后，stack和queue实际上也比较容易去实现

对于这两个容器来说，他们与vector和list有着本质上的区别，因为vector和list实际上是基础的容器，分别对应着顺序表和链表这两个基础的逻辑结构，而对于stack和queue来说，他们可以理解为拥有特定性质的基础容器，可以暂且理解为相对高级的容器

这一点我们可以从他们的模板类声明中得到验证

![](https://s2.loli.net/2023/12/21/xzBDTuQplGqn2ay.png)

![](https://s2.loli.net/2023/12/21/aOe3UynkLYHQ5KF.png)

![](https://s2.loli.net/2023/12/21/WqhGvgfTY5JE2Rx.png)

![](https://s2.loli.net/2023/12/21/IOwlZUD4gMaWk5n.png)

我们可以看到，他们对模板的声明不同

前两个对应的是allocator，这其实是一种空间配置器，是负责封装堆内存管理的对象，本质上来说他是从内存池中分配内存到vector中，在使用的时候也无需配置，因为已经赋予了他缺省值

后两个是container，是容器适配器，他本质上是用另一种容器来初始化这个容器，举个例子，理论上来说，我们可以实现顺序栈和链栈，或者顺序队列和链队列，而他们正是由底层的顺序表或者链表来构成的，因此这个container则代表着实现他们底层的容器

由此我们可以看到实现stack和queue的缺省值是一个名为deque的容器，稍后我们会对他进行介绍，我们在使用stack和queue的时候也可以直接对其进行指定

例如

```c++
stack<int,vector<int>> st;
```

这样我们就创建了一个顺序栈，同理也可以创建顺序队列等不同的容器

## stack和queue的基本使用

| 函数    | 说明               |
| ------- | ------------------ |
| stack() | 构造空栈           |
| empty() | 判空               |
| size()  | 返回栈中元素个数   |
| top()   | 返回栈顶元素的引用 |
| push()  | 将val压栈          |
| pop()   | 将栈顶元素弹出     |

| 函数    | 说明               |
| ------- | ------------------ |
| queue() | 构造空队列         |
| empty() | 判空               |
| size()  | 返回队列中元素个数 |
| front() | 返回队首元素的引用 |
| back()  | 返回队尾元素的引用 |
| push()  | 尾插               |
| pop()   | 头出               |

## stack和queue的模拟实现

由于使用了容器适配器，stack和queue的实现非常简单，只需要提供相应接口调用对应的函数即可

### stack

```C++
namespace xu:
{
    template<typename T, class Container = std::deque<T>>
    class stack
    {
    public:
    	stack() {}
        void push()
        {
            _con.push_back();
        }
    	void pop()
        {
            _con.pop_back();
        }
        T& top()
        {
            return _con.back();
        }
        const T& top() const
        {
            return _con.back();
        }
        size_t size() const
        {
            return _con.size();
        }
        bool empty() const
        {
            return _con.empty();
        }
    private:
    	Container _con;
    };
}
```

queue的的实现几乎与stack一模一样，这里不赘述了，接下来让我们认识一下deque

# deque的基本介绍

前面我们讲了，deque实际上是stack和queue的底层容器，那么他为什么可以代替vector和list呢

deque实际上叫做双端队列，他是一种双开口的“连续”空间的数据结构，双开口的意思是就是他可以在头尾两端进行插入和删除，且时间复杂度为O(1)，比vector的头插效率高，比list的空间利用率高

但实际上deque的空间并不连续，他长下面这样

![](https://s2.loli.net/2023/12/21/5RerF8kaLyf4U1Z.png)

现在让我来解释一下这张图，中控器实际上是一张顺序表，里面存放着指向各个缓冲区（buffer）的指针，也就是指针数组，每一个buffer也是一个数组，大小相同，存放真正的数据

但是他本质上是分段连续的，为了能够实现整体连续的效果，迭代器的设计实现就相对复杂，每一个迭代器都指向一个buffer，其中有四个指针，分别是，cur指向当前buffer的使用到什么位置了，first指向buffer的开头，last指向buffer的结尾，node是一个二级指针，指向中控器的自身，因为一旦这个buffer用完之后再尾插数据需要寻找下一个buffer，就要往回找到中控器才能实现

当deque头插时，实际上是从数据头部从后往前插入，这也是deque从中间开始存数据的好处，可以实现O(1)的头插

## deque的优点

与vector比较，头部插入删除时，不需要移动元素，效率非常高，在扩容时，也不需要搬移大量元素

与list比较，deque的底层是连续空间，空间利用率较高，不需要频繁开辟空间

## deque的缺点

deque不适合任何形式的遍历和随机访问，因为deque的迭代器需要频繁检测是否存储到buffer的边界，导致效率低下，因此在实际场景中，我们使用vector和list较多，甚至可以认为，deque就是专门为了实现stack和queue而专门设计的容器

因为对于stack和queue来说，他们不需要进行遍历，只需要在固定的一端或者两端进行操作即可，因此stack和queue完美发扬了deque的优点而避开了他的缺点

