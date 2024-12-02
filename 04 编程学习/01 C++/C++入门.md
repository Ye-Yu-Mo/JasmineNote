---
title: C++入门
date: 2023-07-25 16:00:06
tags: [C++]
categories: [C++]
---

# 1. C++关键字
C++共计63个关键字,其中c语言有32个
```cpp
asm do if return try continue
auto double inline short typedef for
bool dynamic_cast int signed typename throw
break else long sizeof typeid public
case enum mutable static union wchar_t
catch explicit namespace static_cast unsigned default
char export new struct using friend
class extern operator switch virtual register
const false private template void true
const_cast float protected this volatile while
delete goto reinterpret_cast
```
# 2. 命名空间
在C/C++中，变量、函数和后面要学到的类都是大量存在的，这些变量、函数和类的名称将都存在于全局作用域中，可能会导致很多冲突。使用命名空间的目的是对标识符的名称进行本地化以**避免命名冲突或名字污染**，namespace关键字的出现就是针对这种问题的。

>例如
>```c
>#include<stdio.h>
>#include<stdlib.h>
>
>int rand = 1;
>int main()
>{
>	printf("%d\n",rand);
>	return 0;
>}
>```
>结果如下
>![结果](https://img-blog.csdnimg.cn/96b85f6b111f46d2a638f087d696dbc6.png)
## 2.1 命名空间的定义
命名空间需要用到namespace关键字，类似于struct结构体的定义，{}内为命名空间的成员
```cpp
namespace xu//xu即为命名空间的名字
{
	//1.可定义 变量 函数 类型
	int rand = 1;
	
	void swap(int* a, int* b)
	{
		int tmp = *a;
		*a = *b;
		*b = tmp;
	}
	
	struct node
	{
		struct node* next;
		int val;
	};
}

namespace xu0
{
	int a;
	//2.命名空间可以嵌套定义
	namespace xu1
	{
		int a;
	}
}

namespace xu0
{
	//同名的命名空间自动合并(同一个工程中)
	int b;
}
```

## 2.2 命名空间的使用
首先直接使用肯定是不行的
```cpp
namespace xu
{
	int a = 1;
	
	void swap(int* a, int* b)
	{
		int tmp = *a;
		*a = *b;
		*b = tmp;
	}
	
	struct node
	{
		struct node* next;
		int val;
	};
}
int main()
{
	printf("%d\n",a);
	return 0;
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7e4fc635314d450e96eef90ff571aa71.png)
想要正常使用一般有三种方法
### 2.2.1 加命名空间名称以及作用域限定符
```cpp
int main()
{
	printf("%d",xu::a);
	//相当于告诉编译器，哥们我要用xu中的a变量，别找错了
	return 0;
}
```
### 2.2.2 使用using将命名空间中的某个成员引入
```cpp
using xu::a;
int main()
{
	printf("%d",a);
	return 0;
}
```
一般用于命名空间完全展开后存在命名冲突，但某个名称使用多次且不冲突时，比较方便
### 2.2.3 使用using namespace将命名空间展开
```cpp
using namespace xu
int main()
{
	printf("%d",a);
	return 0;
}
```
# 3. C++的输入和输出
## Hello Wrold!
```cpp
#include<iostream>
using namespace std;
int main()
{
	cout<<"Hello World!<<endl;
	return 0;
}
```
> 说明
> 1. std是**C++标准库的命名空间名称**，C++将标准库的定义实现都放到这个命名空间中
> 2. 使用cout标准输出对象(控制台)和cin标准输入对象(键盘)时，必须包含< iostream >头文件，这个头文件是C++的输入输出流的头文件以及按命名空间使用方法使用std。
> 3. cout和cin是全局的流对象，endl是特殊的C++符号，表示换行输出，他们都包含在包含< iostream >头文件中。endl 的含义简单来说就是结束输出+换行
> 4.**<<是流插入运算符，>>是流提取运算符**
> 5. 使用C++输入输出更方便，不需要像printf/scanf输入输出时那样，需要手动控制格式。
> C++的输入输出可以自动识别变量类型。
> 6. 实际上cout和cin分别是ostream和istream类型的对象，>>和<<也涉及运算符重载等知识
```cpp
#include<iostream>
using namespace std;

int main()
{
	int integer;
	double fraction;
	char ch;

	cin>>integer;
	cin>>fraction>>ch;

	cout<<integer<<endl;
	cout<<fraction<<"  "<<ch<<endl;
	return 0;
}
```
>这里的cin和cout看上去很“智能”，因为相比c语言他不需要指定输入输出类型，而是能够自动识别，实际上涉及运算符重载知识
>std这个命名空间的很多重名问题时，为了简化输入输出的形式，我们一般单独授权就行 using std::cin

# 4. 缺省参数
## 4.1 概念
缺省参数是**声明或定义函数时_以声明居多**为函数的参数**指定一个缺省值**。在调用该函数时，如果没有指定实参则采用该形参的缺省值，否则使用指定的实参

```cpp
#include<iostream>
using namespace std;
void fun(int a = 114)
{
	cout<<a<<endl;
}
int main()
{
	fun();
	fun(514);
	return 0;
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/719368f14cd44b3897361ca756c3381e.png)
## 4.2 分类
### 4.2.1 全缺省参数
```cpp
void Func(int a = 11, int b = 45, int c = 14)
 {
     cout << "a = " << a <<endl;
     cout << "b = " << b <<endl;
     cout << "c = " << c <<endl;
 }
```
### 4.2.2 半缺省参数
```cpp
void Func(int a , int b = 114, int c = 514)
 {
     cout << "a = " << a <<endl;
     cout << "b = " << b <<endl;
     cout << "c = " << c <<endl;
 }
```
>注意
>1. 缺省参数**从右往左**依次赋值，不允许间隔
>2. 缺省参数只能出现在声明或定义，不能同时出现
>3. 缺省参数只能是常量或全局变量
>4. c语言编译器不支持缺省参数
# 5. 函数重载
## 5.1 概念
**函数重载**：是函数的一种特殊情况，C++允许在**同一作用域中**声明几个功能类似的**同名函数**，这些同名函数的**形参列表(参数个数 或 类型 或 类型顺序)不同**，常用来处理实现功能类似数据类型不同的问题
### 5.1.1 参数类型不同
```cpp
int Add(int left, int right)
{
	cout << "int" << endl;
	return left + right;
}
double Add(double left, double right)
{
	cout << "double" << endl;
	return left + right;
}
```
### 5.1.2 参数个数不同
```cpp
void fun1()
{
	cout << "fun1()" << endl;
}
void fun2(int a)
{
	cout << "fun2(int a)" << endl;
}
```
### 5.1.3 类型顺序不同
```cpp
void f(int a, char b)
{
 	cout << "f(int a,char b)" << endl;
}
void f(char b, int a)
{
 	cout << "f(char b, int a)" << endl;
}
```
## 5.2 C++支持函数重载的原理
在C/C++中程序要运行需要经过，**预处理，编译，汇编，链接**将源文件编译成目标文件，再进行链接后形成可执行程序

在C++中，编译器会将函数名进行修饰，不同的编译器修饰的规则也各有不同，具体可以参考[C/C++调用](http://t.csdn.cn/r9gqi)

修饰后的函数名称自然不同，编译器也就能区分你调用的是哪个函数了

这里只是进行简单的形象化理解，具体还需要参考专业资料

# 6. 引用
## 6.1 概念
引用在C++中是给已经存在的变量取个别名，编译器不会为变量开辟空间，会和变量共用空间
```cpp
void TestRef()
{
    int a = 10;
    int& ra = a;//定义 a的别名为ra
    printf("%p\n", &a);
    printf("%p\n", &ra);
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/461e3d4c32294e1693bd9feeab145ab4.png)
两个名称取地址后的结果一样，代表两个名字指向同一块空间
>**引用的类型和变量的类型必须相同**

## 6.2 特性
> 1. 定义引用时必须初始化
> 2. 一个变量可以拥有多个别名
> 3. 一个别名只能对应一个变量，且不可改变
> 4. 常量和常变量不可引用
> 5. 类型不同会报错

## 6.3 应用
### 6.3.1 做参数
对比没有引用的时候，需要使用指针来对函数外的变量进行操作
```cpp
void swap(int* a, int* b)
{
	int tmp = *a;
	*a = *b;
	*b = tmp;
}
```
```cpp
void swap(int& a, int&b)
{
	int tmp = a;
	a = b;
	b = tmp;
}
```
这里是对交换值的简单理解，其实可以延申到对栈，链表中的应用，会方便很多
### 6.3.2 做返回值
>这里主要进行形象化的描述，更加专业的表述需要查看具体的书籍。
>
>给一串示例代码
```cpp
#include<iostream>
using namespace std;
int& add(int a, int b)
{
	int n = a+b;
	return n;
}
int main()
{
	int ret = add(1,2);
	return 0;
}
```
这里简单写了一个相加的函数，在add的n返回之前，会先将n赋值给一个临时变量tmp，再把tmp起个别名返回给ret，那么很容易出现bug的一个地方时，再返回给ret之前，内存空间变化了，但是tmp的别名所指向的位置没有变化，就会出现随机值

## 6.4 传值和传引用的效率问题
 以值作为参数或者返回值类型，在传参和返回期间，函数不会直接传递实参或者将变量本身直接返回，而是传递实参或者返回变量的一份临时的拷贝，因此用值作为参数或者返回值类型，效率是非常低下的，尤其是当参数或者返回值类型非常大时，效率就更低
所以在传递大量数据时，可以优先考虑传引用
## 6.5 引用和指针的区别
引用是一个别名，没有独立空间，和引用的变量共用空间，但其实在C++中，引用的底层实现仍然是使用指针的方式来实现的，我们可以通过汇编代码看到
```cpp
int main()
{
	int a = 10;
	int& ra = a;
	ra = 20;
	int* pa = &a;
	*pa = 20;
	return 0;
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/daa236233b5f4c95841115a433ac12a3.png)
红色圈出来的汇编代码是完全相同的
>以下是列举的一些指针和引用的不同点
>1. 引用概念上定义一个变量的别名，指针存储一个变量地址。
>2. 引用在定义时必须初始化，指针没有要求
>3. 引用在初始化时引用一个实体后，就不能再引用其他实体，而指针可以在任何时候指向任何一个同类型实体
>4. 没有NULL引用，但有NULL指针
>5. 在sizeof中含义不同：引用结果为引用类型的大小，但指针始终是地址空间所占字节个数(32位平台下占4个字节)
>6. 引用自加即引用的实体增加1，指针自加即指针向后偏移一个类型的大小
>7. 有多级指针，但是没有多级引用
>8. 访问实体方式不同，指针需要显式解引用，引用编译器自己处理
>9. 引用比指针使用起来相对更安全
# 7. 内联函数 #inline

## 7.1 概念
以**inline修饰**的函数叫做内联函数,**编译时**C++编译器会在**调用内联函数的地方展开**,没有函数调用建立栈帧的开销，内联函数提升程序运行的效率。


```cpp
#include<iostream>
using namespace std;

int add(int a, int b)
{
	return a + b;
}
int main()
{
	add(1, 2);
	return 0;
}
```

不加inline时的汇编代码如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/f9a44d0f7cd5435a9be10afe4e34a2cf.png)
这里就call到了add函数的地址，相当于调用了add函数

```cpp
#include<iostream>
using namespace std;

inline int add(int a, int b)
{
	int c = a + b;
	return c;
}
int main()
{
	add(1, 2);
	return 0;
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/681885f5e8f9400a86ea3860a2d36638.png)
这里看到编译器并没有call add函数，而是直接把add函数的内容放在了主函数中
>注意
>1. 在release模式下查看汇编代码是否存在call add
>2. 在debug模式下需要进行设置，否则不会展开
>![在这里插入图片描述](https://img-blog.csdnimg.cn/182e6d5e75bc4ca2885df1ea8b0cc7a1.png)
>![在这里插入图片描述](https://img-blog.csdnimg.cn/4b032415460140508c176032ef13527d.png)

## 7.2 特性
>1. inline是一种以空间换时间的做法，如果编译器将函数当成内联函数处理，在编译阶段，会用函数体替换函数调用，缺陷：可能会使目标文件变大，优势：少了调用开销，提高程序运行效率。
>2. inline对于编译器而言只是一个请求，不同编译器关于inline实现机制可能不同，一般建议：将函数规模较小(即函数不是很长，具体没有准确的说法，取决于编译器内部实现)、不是递归、且频繁调用的函数采用inline修饰，否则编译器会忽略inline特性。
>3. inline不建议声明和定义分离，分离会导致链接错误。因为inline被展开，就没有函数地址了，链接就会找不到

# 8. auto（C++11）
## 8.1 由来
由于类型名称越来越复杂，容易记错，难以拼写，而且含义也不好直接理解，但是如果仅仅使用typedef的话就更难理解指针的级数，仍然需要频繁查看源代码
## 8.2 使用
### 8.2.1 auto与指针，引用结合
```cpp
int main()
{
    int x = 10;
    auto a = &x;
    auto* b = &x;
    auto& c = x;
    cout << typeid(a).name() << endl;
    cout << typeid(b).name() << endl;
    cout << typeid(c).name() << endl;
    *a = 20;
    *b = 30;
     c = 40;
    return 0;
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/29da26f762e44fa4a21e2f3af86a436e.png)
通过上面可以看到auto和auto*是没有区别的，但是引用的时候必须加上&

### 8.2.2 在同一行定义多个变量
在同一行定义不同变量时，若使用auto，那么变量的类型必须相同，否则会编译失败
![在这里插入图片描述](https://img-blog.csdnimg.cn/d2f5b575d27e4de7a36d8f5fe745381a.png)

## 8.3 auto不能使用的场景
>1. 不能作为函数的参数
>```cpp
>void TestAuto(auto a)
>{}
>```
>![在这里插入图片描述](https://img-blog.csdnimg.cn/18fd37bbde844e24b7e6accc0170bb09.png)
>2. 不能用来声明数组
>3. 避免与C++98中的auto发生混淆，C++11只保留了auto作为了些指示符的用法
# 9. 范围for循环
在C语言或者C++98中要使用for循环一般采用如下形式
```cpp
void testFor()
{
	int arr[] = {1,1,4,5,1,4};
	for(int i = 0; i<sizeof(arr)/sizeof(arr[0]);i++)
		arr[i] *= 2;
}
```
对于一个**有范围的集合**，说明循环的范围是冗余的，因为只需要从头到尾遍历即可，由此引出C++11基于范围的for循环，for的括号内由 “ : ” 分为两部分，第一部分是范围用于迭代的代表的变量，第二部分是集合
例如
```cpp
void testFor()
{
	int arr[] = {1,1,4,5,1,4};
	for(auto e : arr)
	{
		e *= 2;
	}
}
```
在这种范围for循环中依然可以使用break，continue
## 9.2 范围for使用的条件
### 9.2.1 范围for的迭代范围是确定的
数组，从第一个到最后一个元素的范围
类，提供begin和end的方法，begin和end就是for循环迭代的范围

# 10. 空指针nullptr
为了防止野指针的问题，我们引入了空指针NULL，实际上NULL，是一个宏，在C的头文件stddef.h中
```c
#ifndef NULL
#ifdef __cplusplus
#define NULL   0
#else
#define NULL   ((void *)0)
#endif
#endif
```
实际上NULL是一个字面常量0，或者是一个无类型指针（void*）的常量
那本意是想定义为一个没有指向的指针，现在用0来代替，实际上就给日常的程序带来一定的麻烦，因为每次都要转为（void*）0

>1. 在使用nullptr时，不需要头文件，因为nullptr在C++11中为新关键字
>2. C++11中，sizeof(nullptr)和sizeof((void*)0)相同
>3. 建议使用nullptr来代表空指针
