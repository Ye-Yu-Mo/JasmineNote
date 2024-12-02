---
title: vector的简单模拟实现
date: 2023-12-12 19:02:39
tags: [C++ ,STL,vector]
categories: [C++ ,STL]
---

# vector的简单模拟实现

由于vector支持各类容器和数据类型，还有内存池等相关部分，这里为了方便初学者学习，不涉及模板的高级知识，只使用基础的C++知识，实现简单的vector的功能

## 头文件概览

```cpp
namespace xu {
	template <class T> // 模板类
	class vector {
	public:
        // 迭代器部分
		typedef T* iterator;
		typedef const T* const_iterator;
		iterator begin();
		iterator end();
		const_iterator cbegin();
		const_iterator cend();

        // 构造函数与析构函数
		vector();
		vector(int n, const T& value = T());
		template<class InputIterator>
		vector(InputIterator first, InputIterator last);
		vector(vector<T>& v);
		vector<T>& operator= (vector<T> v);
		~vector();
            
        // 内存相关
		size_t size() const;
		size_t capacity() const;
		void reserve(size_t n);
		void resize(size_t n, const T& value = T());

        // 增删查改
		T& operator[](size_t pos);
		const T& operator[](size_t pos) const;
		void push_back(const T& x);
		void pop_back();

		bool empty();
		void swap(vector<T>& v);
		iterator insert(iterator pos, const T& x);
		iterator erase(iterator pos);

	private:
        // 这里为了省略构造函数的初始化列表而引入了缺省值
		iterator _start = nullptr;
		iterator _finish = nullptr;
		iterator _EndOfStorage = nullptr;
	};
}
```

## 迭代器

```cpp
		typedef T* iterator;
		typedef const T* const_iterator;
		iterator begin()
		{
			return _start;
		}
		iterator end()
		{
			return _finish;
		}
		const_iterator cbegin()
		{
			return _start;
		}
		const_iterator cend()
		{
			return _finish;
		}
```

> 这里使用了原生指针作为模板的迭代器，还有rbegin、rend没有实现，也相对比较简单

## 构造函数与析构函数

```cpp
		vector()
		{}
		vector(int n, const T& value = T())
		{
			reserve(n);
			while (n--)
				push_back(T);
		}
		template<class InputIterator>
		vector(InputIterator first, InputIterator last)
		{
			for (auto& i = first; i < last; i++)
				push_back(*i);
		}
		vector(vector<T>& v)
		{
			reserve(v.capacity());
			for (const auto& e : v)
				push_back(e);
		}
		vector<T>& operator= (vector<T> v)
		{
			swap(v);
			return *this;
		}
		~vector()
		{
			delete[] _start;
			_start = _finish = _EndOfStorage = nullptr;
		}
```

> 1. 省略了初始化列表，放在了定义的缺省值中
> 2. 由于实现了reserve函数，极大简化了内存申请的相关操作
> 3. 在重载=时，由于v是拷贝构造对象，其内容可以直接使用，因此直接用swap交换其内容即可

## 内存相关

```cpp
		size_t size() const
		{
			return _finish - _start;
		}
		size_t capacity() const
		{
			return _EndOfStorage - _start;
		}
		void reserve(size_t n)
		{
			if (n <= capacity())
				return;
			size_t old_size = size();
			T* tmp = new T[n];
			for (int i = 0; i < size(); i++)
			{
				tmp[i] = _start[i];
			}
			delete[] _start;
			_start = tmp;
			_finish = _start + old_size;
			_EndOfStorage = _start + n;
			
		}
		void resize(int n, const T& value = T())
		{
			if (size() > n)
			{
				_finish = _start + n;
				return;
			}
			reserve(n);
			while (_finish != _start + n)
			{
				*(_finish) = T;
				_finish++;
			}
		
		}
```

>reserve中需要注意一点，在重新申请空间之后，需要注意更新变量的顺序，否则会出现 _finish 为空指针的情况，这里处理方式有两种，一是按照 _finish _start _EndOfStorage 的顺序初始化，二是先保存下原先的数据大小，在扩容后重新赋值即可

## 增删查改

```cpp
		T& operator[](size_t pos)
		{
			return *(_start + pos);
		}
		const T& operator[](size_t pos) const
		{
			return *(_start + pos);
		}

		void push_back(const T& x)
		{
			if (_EndOfStorage == _finish)
			{
				size_t old_size = size();
				size_t newcapacity = capacity() == 0 ? 4 : capacity() * 2;
				reserve(newcapacity);
			}
			(*_finish) = x;
			_finish++;
		}
		void pop_back()
		{
			assert(!empty());
			delete[] _finish;
			_finish--;
		}

		bool empty()
		{
			return size() == 0;
		}
		void swap(vector<T>& v)
		{
			std::swap(_start, v._start);
			std::swap(_finish, v._finish);
			std::swap(_EndOfStorage, v._EndOfStorage);
		}
		iterator insert(iterator pos, const T& x)
		{
			if (_EndOfStorage == _finish)
			{
				size_t len = pos - _start;
				reserve(capacity() + 1);
				pos = _start + len;
			}
			for (iterator i = _finish - 1; i != pos; i--)
			{
				*(i + 1) = *i;
			}
			*(pos) = x;
			_finish++;
			return pos;

		}
		iterator erase(iterator pos)
		{
			assert(!empty());
			while (pos != _finish)
			{
				*(pos) = *(pos + 1);
				pos++;
			}
			_finish--;
			return pos;
		}

```

> 这里需要注意insert会出现和reserve一样的问题，数据的新旧，在扩容之后，由于pos仍然指向之前内容，需要记录和 _start 的相对位置，在扩容之后恢复即可

## 整体代码

```cpp
#include<iostream>
#include<cassert>


namespace xu {
	template <class T>
	class vector {
	public:
		typedef T* iterator;
		typedef const T* const_iterator;
		iterator begin()
		{
			return _start;
		}
		iterator end()
		{
			return _finish;
		}
		const_iterator cbegin()
		{
			return _start;
		}
		const_iterator cend()
		{
			return _finish;
		}

		vector()
		{}
		vector(int n, const T& value = T())
		{
			reserve(n);
			while (n--)
				push_back(T);
		}
		template<class InputIterator>
		vector(InputIterator first, InputIterator last)
		{
			for (auto& i = first; i < last; i++)
				push_back(*i);
		}
		vector(vector<T>& v)
		{
			reserve(v.capacity());
			for (const auto e : v)
				push_back(e);
		}
		vector<T>& operator= (vector<T> v)
		{
			swap(v);
			return *this;
		}
		~vector()
		{
			delete[] _start;
			_start = _finish = _EndOfStorage = nullptr;
		}

		size_t size() const
		{
			return _finish - _start;
		}
		size_t capacity() const
		{
			return _EndOfStorage - _start;
		}
		void reserve(size_t n)
		{
			if (n <= capacity())
				return;
			size_t old_size = size();
			T* tmp = new T[n];
			for (int i = 0; i < size(); i++)
			{
				tmp[i] = _start[i];
			}
			delete[] _start;
			_start = tmp;
			_finish = _start + old_size;
			_EndOfStorage = _start + n;
			
		}
		void resize(int n, const T& value = T())
		{
			if (size() > n)
			{
				_finish = _start + n;
				return;
			}
			reserve(n);
			while (_finish != _start + n)
			{
				*(_finish) = T;
				_finish++;
			}
		
		}

		T& operator[](size_t pos)
		{
			return *(_start + pos);
		}
		const T& operator[](size_t pos) const
		{
			return *(_start + pos);
		}

		void push_back(const T& x)
		{
			if (_EndOfStorage == _finish)
			{
				size_t old_size = size();
				size_t newcapacity = capacity() == 0 ? 4 : capacity() * 2;
				reserve(newcapacity);
			}
			(*_finish) = x;
			_finish++;
		}
		void pop_back()
		{
			assert(!empty());
			delete[] _finish;
			_finish--;
		}

		bool empty()
		{
			return size() == 0;
		}
		void swap(vector<T>& v)
		{
			std::swap(_start, v._start);
			std::swap(_finish, v._finish);
			std::swap(_EndOfStorage, v._EndOfStorage);
		}
		iterator insert(iterator pos, const T& x)
		{
			if (_EndOfStorage == _finish)
			{
				size_t len = pos - _start;
				reserve(capacity() + 1);
				pos = _start + len;
			}
			for (iterator i = _finish - 1; i != pos; i--)
			{
				*(i + 1) = *i;
			}
			*(pos) = x;
			_finish++;
			return pos;

		}
		iterator erase(iterator pos)
		{
			assert(!empty());
			while (pos != _finish)
			{
				*(pos) = *(pos + 1);
				pos++;
			}
			_finish--;
			return pos;
		}

	private:
		iterator _start = nullptr;
		iterator _finish = nullptr;
		iterator _EndOfStorage = nullptr;
	};
}
```

==**感谢支持，如果你发现文章中有任何不严谨或者需要补充的部分，欢迎在评论区指出**==
