---
title: C/C++static关键字详解
date: 2024-09-30 12:03:10
tags:
  - C++
categories:
  - C++
---

static是一个非常重要的关键字，为什么呢，用法又多，说不定哪里来一下就给人整懵了，而且还比较复杂，有必要专门记录一下static的用法

static的意思是静态的

一开始在学习C语言时，我们时用它来控制变量和函数的作用域，也就是使用范围，后面学习到进程地址空间，才了解到他也改变了存储的区域

我们知道进程地址空间可以大致分为栈区、堆区、静态区

诶这个静态区是不是眼熟，没错

static修饰的变量都存在静态区，一直到程序结束时才会释放

额外bb一句，所有的常量也存在静态区

简单说，static修饰的对象无非就是变量和函数，但是变量和函数放的位置不一样，static的作用也不一样

## C语言中的static

C语言中static一共有三个用法

+ 修饰局部变量->静态局部变量
+ 修饰全局变量->静态全局变量
+ 修饰函数->静态函数

### 局部变量

1. 在局部变量中static修饰的变量只初始化一次，在之后调用这个函数的时候保留原来的状态
2. 有点类似于全局函数的意思，但是延长了局部变量的生命周期，但是他的作用域还是在函数范围内
3. 修改了他的存储位置，从栈区改到了静态区
4. 如果static变量没有赋初值，则自动初始化为0，这一点也是类似于全局变量的

### 全局变量和函数

1. 修改了全局变量的作用范围，以前的全局是真全局，整个项目中都能用（使用extern外部声明），生命周期贯穿程序
2. static修饰之后只能在内部链接，说人话就是只能在这个文件中使用，别的文件extern了也不行

这里对函数和全局变量的限制是相同的，目前还没有什么区别

## C++中的static

C++是兼容C语言的，所以C语言中的static的用法中C++是一样的

除了修饰上面的三个，static还可以修饰类内成员，包括类内成员变量和类内成员函数分别变成静态成员变量和静态成员函数

重点特性

1. 静态成员是所有的该类实例化之后的实例所共享，不属于某个具体的实例
2. 静态成员变量必须在类外定义，定义时不添加static关键字，相当于在类内是生命，类外是初始化
3. 静态成员函数没有this指针，不能访问任何非静态成员，无论是变量还是函数
4. 静态成员也有访问级别的限制，public、protected、private，可以有返回值

### 类内声明，类外定义

```cpp
class A{
private:
    static int _a;
};
int A::_a = 114;
```

### 访问静态成员

```cpp
class A{
public:
    static int _a;
public:
    static void int func(){
        
    }
};

int main()
{
    A a;
    a._a;
    A::_a;
    A::func();
    A()._k;
    return 0;
}
```

上面的三种方式对于静态成员变量和静态成员函数都是适用的

注意：静态成员函数虽然在类内，但是无法调用其他非静态成员函数，不能使用静态成员变量，但是可以使用其他的静态成员函数

