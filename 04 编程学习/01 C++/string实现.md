---
title: string类的模拟实现
date: 2023-12-08 14:38:24
tags: [C++,STL,string]
categories: [C++,STL]
---

# string模拟实现

## 头文件

```cpp
#pragma once
#define _CRT_SECURE_NO_WARNINGS 1
#include<iostream>
#include<cstring>
#include<cassert>

#define npos -999

namespace xu
{
	class string
	{
        //流插入流提取是左操作符，需要声明友元函数
		friend std::ostream& operator<<(std::ostream& _cout, const xu::string& s);
		friend std::istream& operator>>(std::istream& _cin, xu::string& s);

	public:
   		//使用原生指针作为迭代器
		typedef char* iterator;

        //构造函数与析构函数
		string(const char* str = "");
		string(const string& s);
		string& operator=(const string& s);
		string& operator=(const char* str);
		~string();

        //迭代器获取
		iterator begin();
		iterator end();

        //string基本操作
		void push_back(char c);
		string& operator+=(char c);
		string& operator+=(const char* str);
		string& operator+=(const string& s);
		void clear();
		void swap(string& s);
		const char* c_str()const;

        //内存操作
		size_t size()const;
		size_t capacity()const;
		bool empty()const;
		void resize(size_t n, char c = '\0');
		void reserve(size_t n);

        //读取操作
		char& operator[](size_t index);
		const char& operator[](size_t index)const;

        //大小比较
		bool operator<(const string& s);
		bool operator==(const string& s);
		bool operator<=(const string& s);
		bool operator>=(const string& s);
		bool operator>(const string& s);
		bool operator!=(const string& s);

        //查找、插入、删除
		size_t find(char c, size_t pos = 0) const;
		size_t find(const char* s, size_t pos = 0) const;
		string& insert(size_t pos, char c);
		string& insert(size_t pos, const char* str);
		string& erase(size_t pos = 0, size_t len = npos);
	private:
		char* _str;
		size_t _capacity;
		size_t _size;
	};
}
```

## 源文件

```cpp
#include"MyString.h"

xu::string::string(const char* str)
{
	assert(str != nullptr);
	_size = strlen(str);
	_capacity = _size <= 17 ? 17 : _size;
	_str = new char[_capacity];
	strcpy(_str, str);
}

xu::string::string(const string& s)
{
	_size = s._size;
	_capacity = _size <= 17 ? 17 : _size;
	_str = new char[_capacity];
	strcpy(_str, s._str);
}

xu::string& xu::string::operator=(const xu::string& s)
{
	if (_capacity < s._capacity)
	{
		_str = new char[s._capacity];
		_capacity = s._capacity;
	}
	_size = s._size;
	strcpy(_str, s._str);
	return *this;
}

xu::string& xu::string::operator=(const char* str)
{
	size_t size = strlen(str) + 1;
	if (_capacity < size)
	{
		_str = new char[size];
		_capacity = size;
	}
	_size = size;
	strcpy(_str, str);
	return *this;
}

xu::string::~string()
{
	_capacity = 0;
	_size = 0;
	delete[] _str;
}

xu::string::iterator xu::string::begin()
{
	return _str;
}

xu::string::iterator xu::string::end()
{
	return _str + _size;
}

void xu::string::push_back(char c)
{
	reserve(_capacity + 1);
	_str[_size++] = c;
	_str[_size] = '\0';
}


xu::string& xu::string::operator+=(char c)
{
	push_back(c);
	return *this;
}

xu::string& xu::string::operator+=(const char* str)
{
	size_t len = strlen(str);
	for (size_t i = 0; i < len; i++)
		push_back(str[i]);
	return *this;
}

size_t xu::string::size() const
{
	return _size;
}

size_t xu::string::capacity() const
{
	return _capacity;
}

bool xu::string::empty() const
{
	return _size == 0;
}

void xu::string::resize(size_t n, char c)
{
	if (n <= _size)
	{
		_str[n] = '\0';
		_size = n;
	}
	reserve(n);
	for (size_t i = _size; i < n; i++)
		push_back(c);
	_str[n] = '\0';
}

void xu::string::reserve(size_t n)
{
	if (_capacity >= n)
		return;
	while (_capacity < n)
	{
		_capacity *= 2;
	}
	_str = (char*)realloc(_str, _capacity);
}

xu::string& xu::string::operator+=(const string& s)
{
	for (size_t i = 0; i < s._size; i++)
		push_back(s._str[i]);
	return *this;
}

void xu::string::clear()
{
	_str[0] = '\0';
	_size = 0;
}

void xu::string::swap(string& s)
{
	std::swap(_str, s._str);
	std::swap(_size, s._size);
	std::swap(_capacity, s._capacity);
}

const char* xu::string::c_str() const
{
	return _str;
}

char& xu::string::operator[](size_t index)
{
	return _str[index];
}

const char& xu::string::operator[](size_t index) const
{
	return _str[index];
}

bool xu::string::operator<(const xu::string& s)
{
	for (size_t i = 0;_str[i]!='\0'||s._str[i]!='\0'; i++)
	{
		if (_str[i] < s._str[i])
			return true;
		else if (_str[i] > s._str[i])
			return false;
	}
	return false;
}

bool xu::string::operator==(const xu::string& s)
{
	for (size_t i = 0; _str[i] != '\0' || s._str[i] != '\0'; i++)
	{
		if (_str[i] != s._str[i])
			return false;
	}
	return true;;
}

bool xu::string::operator<=(const xu::string& s)
{
	return (*this < s) || (*this == s);
}

bool xu::string::operator>=(const xu::string& s)
{
	return !(*this < s);
}

bool xu::string::operator>(const xu::string& s)
{
	return !(*this <= s);
}

bool xu::string::operator!=(const xu::string& s)
{
	return !(*this == s);
}

size_t xu::string::find(char c, size_t pos) const
{
	assert(pos < _size);
	for (size_t i = pos; i < _size; i++)
	{
		if (_str[i] == c)
			return i;
	}
	return -1;
}

size_t xu::string::find(const char* c, size_t pos) const
{
	assert(pos < _size);
	char* ret = strstr(_str, c);
	if (ret == nullptr)
		return -1;
	return ret - _str;
}

xu::string& xu::string::insert(size_t pos, char c)
{
	assert(pos <= _size);
	reserve(_size + 1);
	for (size_t i = _size; i >= pos; i--)
	{
		_str[i + 1] = _str[i];
	}
	_str[pos] = c;
	_size++;
	return *this;
}

xu::string& xu::string::insert(size_t pos, const char* c)
{
	assert(pos <= _size);
	size_t len = strlen(c);
	reserve(_size + len);
	for (size_t i = _size; i >= pos; i--)
	{
		_str[i + len] = _str[i];
	}
	for (size_t i = 0; i < len; i++)
		_str[pos + i] = c[i];
	_size += len;
	return *this;
}

xu::string& xu::string::erase(size_t pos, size_t len)
{
	assert(pos <= _size);
	if (len == npos)
	{
		_str[pos] = '\0';
		_size = pos;
		return *this;
	}
	for (size_t i = 0; _str[i + pos + len] != '\0'; i++)
	{
		_str[i + pos] = _str[i + pos + len];
	}
	_size -= len;
	_str[pos + len] = '\0';
	return *this;
}

std::ostream& xu::operator<<(std::ostream& _cout, const xu::string& s)
{
	std::cout << s._str;
	return _cout;
}

std::istream& xu::operator>> (std::istream& _cin, xu::string& s)
{
	std::cin >> s._str;
	s.reserve(strlen(s._str));
	s._size = strlen(s._str);
	return _cin;
}
```

==**如果你在阅读和使用过程中发现了什么问题或者发现可以简化的部分，欢迎留言讨论！感谢支持**==
