---
title: 红黑树封装map和set
date: 2024-04-12 20:24:58
tags: [C++,数据结构,红黑树,map,set]
categories: [C++]
---

## 封装的一些小问题

### 封装要实现什么

一般来说封装需要实现[[map和set]]的迭代器，包括普通迭代器和const迭代器，其次是++和--操作，还有判断是否相等的重载

除此之外我们需要实现[[map和set]]的特性，例如set的值不允许修改，map的key不允许修改

### 伏笔一

set的构造参数只有一个，而map的构造参数有两个，我们怎么只用一个数据结构（红黑树）来封装两个容器呢

RBTree的[[模板]]参数前两个是K和T，K表示key关键字类型，T表示其中的数据类型

当用来构造map时刚刚好，一个用来存key，另一个用来存数据，这里我们考虑使用pair，分别存入const K和V，用来确保K不被修改

当用来构造set时，K和T是相同的，我们都传入K即可

### 伏笔二

RBTree的[[模板]]参数里面有一个KeyOfT，这是一个仿函数，因为map的结构特殊性，是一个pair，所以我们需要用仿函数来自动调用对应的取key中的T的操作，类似于C语言的回调函数，根据数据的类型调用不同的函数

### 伏笔三

在实现Insert函数时，返回值是一个pair，first是Node*，second是一个布尔值

这里的Node*一方面是需要返回给外部调用函数，另一方面是不能返回对应的迭代器和布尔类型，等下我们写到对应代码时再具体解释

## RBTree迭代器

### 基础架构和简单内容实现

```cpp
template<class T>
struct __TreeIterator {
	typedef RBTreeNode<T> Node;
	typedef __TreeIterator<T> Self;
	Node* _node;

	__TreeIterator(Node* node) 
		:_node(node)
	{}

	T& operator*() {
		return _node->_data;
	}

	T* operator->() {
		return &_node->_data;
	}

	bool operator!=(const Self& s) {
		return _node != s._node;
	}

	bool operator==(const Self& s) {
		return _node == s._node;
	}
};
```

### ++的实现

要实现++我们首先要明白++的本质是什么

一般来说迭代器的加减本身是用于寻找下一个（上一个）位置的迭代器，那对于红黑树这个数据结构来讲，他是中序有序的，因此我们要找的就是中序遍历的下一个节点

这里直接想其实是不容易的，我们通过例子来讲解

![image.png](https://s2.loli.net/2024/04/12/fprEHUjK7T2qAPL.png)

第一个例子是，1的下一个节点是6，8的下一个节点是11，17的下一个节点是22，那我们其实就发现，cur的右子树存在，直接走到他的右子树的最左节点即可，那么如果他的右节点不存在呢

第二个例子，6的下一个节点是8，11的下一个节点是13，15的下一个节点是17，22的下一个节点是25，规律不怎么明显，结论是如果右节点不存在则回溯，回溯到孩子是父亲左的第一个祖先节点

这个规律似乎很奇怪

但其实我们只需要想一下中序遍历的顺序：左子树、根、右子树

cur就是根，我们已经访问完了根，想要找他的下一个节点，那根的下一个就是右子树最左节点，如此往复，一直到没有右子树了，接下来寻找的孩子是父亲左的第一个节点，这句话实际上就保证了左子树全部被访问了，因此孩子是父亲左的第一个节点就是下一个cur，也就是根，接下来就需要继续访问右子树了

那类似的--的过程跟++的过程完全相反，找cur的左节点，如果左节点不存在则找孩子是父亲右的第一个节点

```cpp
	Self& operator++() {
		if (_node->_right) {
			Node* cur = _node->_right;
			while (cur->_left) {
				cur = cur->_left;
			}
			_node = cur;
		}
		else {
			Node* cur = _node;
			Node* parent = cur->_parent;
			while (parent && cur == parent->_right) {
				cur = parent;
				parent = parent->_parent;
			}
			_node = parent;
		}
		return *this;
	}
```

这里有个细节是parent为空时我们也认为他找到了，主要是需要包含根节点的情况

## RBTree的修改

```cpp
	typedef __TreeIterator<T> iterator;

	iterator begin() {
		Node* cur = _root;
		while (cur && cur->_left) {
			cur = cur->_left;
		}
		return iterator(cur);
	}

	iterator end() {
		return iterator(nullptr);
	}
```

begin就是返回中序的第一个位置的迭代器，也就是最小的值，就是最左节点

end返回最后一个位置的下一个位置的迭代器，自然就是空指针

## map.h

```cpp
#pragma once
#include"RBTree.h"

namespace xu {
	template<class K, class V>
	class map {
	public:
		struct MapKeyOfT {
			const K& operator() (const pair<K, V>& kv) {
				return kv.first;
			}
		};

		typedef typename RBTree<K, pair<const K, V>, MapKeyOfT>::iterator iterator;
		typedef typename RBTree<K, pair<const K, V>, MapKeyOfT>::const_iterator const_iterator;

		iterator begin() {
			return _t.begin();
		}

		iterator end() {
			return _t.end();
		}

		V& operator[](const K& key) {
			pair<iterator, bool> ret = insert(make_pair(key, V()));
			return ret.first->second;
		}

		pair<iterator, bool> insert(const pair<K, V>& kv) {
			return _t.Insert(kv);
		}
	private:
		RBTree<K, pair<const K, V>, MapKeyOfT> _t;
	};
}
```

1. MapKeyOfT是一个仿函数，用于获取对应K的值
2. typedef typename主要是因为C++的特性原因，我们之前在封装的部分也有遇到过，主要是因为编译器不认识后面这一串东西到底是什么，有可能是内部类，有可能是类型名，也有可能是类成员，因此这里是必须要加上typename的，要告诉编译器这时一个类型名，放心编译
3. 这里方括号的妙用主要是由于pair设计的出色，我在之前介绍[[map和set]]的文章中已经讲解了，这里给出链接[[[map和set]]介绍](http://t.csdnimg.cn/TyFcX)
4. 我们通过const K来限制key不可改变，这里比较简单

## set.h

```cpp
#pragma once
#include"RBTree.h"

namespace xu {
	template<class K>
	class set {
	public:
		struct SetKeyOfT {
			const K& operator(const K& key) {
				return key;
			}
		};

		typedef typename RBTree<K, K, SetKeyOfT>::const_iterator iterator;
		typedef typename RBTree<K, K, SetKeyOfT>::const_iterator const_iterator;

		iterator begin() const {
			return _t.begin();
		}

		iterator end() const {
			return _t.end();
		}

		pair<iterator, bool> insert(const K& key) {
			return _t.Insert(key);
		}
	private:
		RBTree<K, K, SetKeyOfT> _t;
	};
}
```

这里看上去实现也比较简单，但其实里面有一个部分还是比较难以理解的

set本身是不允许被修改的，我们通过限制迭代器来实现这个功能，无论是对于iterator还是const_iterator都使用const_iterator来typedef

但是这里就引出了一个问题，本身iterator和const_iterator是两个类型，当我们在insert里面返回iterator时是需要进行make_pair然后再返回的

然后对于make_pair来说，他是需要对相同类型初始化才能成功构造pair的

```cpp
template<class T1, class T2>
struct pair
{
    template<class U, class V>
    pair(const pair<U, V>& pr)
      :first(pr.first)
      ,second(pr.second)
    {}
}
```

他的构造是长这个样子的，只有当传入的U和V与T1，T2类型能够进行构造时，才能正确拷贝构造

也就是说

如果在RBTree中Insert的返回值类型是`pair<iterator, bool>`时，这个iterator是普通的迭代器，对应到set里的insert的返回值类型也是`pair<iterator, bool>`，这里的iterator是我们上面typedef的，原本的类型应该是const_iterator，因此这两个类型不能进行拷贝构造，就会报错显示`无法从pair<__TreeIterator<T, T&, T*>, bool> 转换为 pair<__TreeIterator<T, const T&, const T*>, bool>`

那对于这里的问题，我们是用到了之前的伏笔三，也就是在RBTree里面Insert的返回值类型是`pair<Node*, bool>`，对于Node*类型，无论是iterator还是const_iterator都可以进行构造，就解决了这个返回值的bug

## 全部代码

```cpp
#pragma once
#include<utility>
#include<iostream>
using namespace std;
// 颜色
enum Color {
	RED,
	BLACK
};

template<class T>
struct RBTreeNode {
	RBTreeNode<T>* _left;
	RBTreeNode<T>* _right;
	RBTreeNode<T>* _parent;

	T _data;

	Color _col;

	RBTreeNode(const T& data)
		:_left(nullptr)
		, _right(nullptr)
		, _parent(nullptr)
		, _data(data)
		, _col(RED) {}
};

template<class T>
struct __TreeIterator {
	typedef RBTreeNode<T> Node;
	typedef __TreeIterator<T> Self;
	Node* _node;

	__TreeIterator(Node* node) 
		:_node(node)
	{}

	T& operator*() {
		return _node->_data;
	}

	T* operator->() {
		return &_node->_data;
	}

	bool operator!=(const Self& s) {
		return _node != s._node;
	}

	bool operator==(const Self& s) {
		return _node == s._node;
	}

	Self& operator++() {
		if (_node->_right) {
			Node* cur = _node->_right;
			while (cur->_left) {
				cur = cur->_left;
			}
			_node = cur;
		}
		else {
			Node* cur = _node;
			Node* parent = cur->_parent;
			while (parent && cur == parent->_right) {
				cur = parent;
				parent = parent->_parent;
			}
			_node = parent;
		}
		return *this;
	}
};


template<class K, class T, class KeyOfT>
class RBTree {
	typedef RBTreeNode<T> Node;
public:
	typedef __TreeIterator<T> iterator;

	iterator begin() {
		Node* cur = _root;
		while (cur && cur->_left) {
			cur = cur->_left;
		}
		return iterator(cur);
	}

	iterator end() {
		return iterator(nullptr);
	}

	pair<Node*, bool> Insert(const T& data) {
		
		if (_root == nullptr) {
			_root = new Node(data);
			_root->_col = BLACK;
			return make_pair(_root, true);
		}

		Node* parent = nullptr;
		Node* cur = _root;
		KeyOfT kot;

		// 平衡二叉树找到插入位置
		while (cur) {
			if (kot(cur->_data) < kot(data)) {
				parent = cur;
				cur = cur->_right;
			} else if (kot(cur->_data) > kot(data)) {
				parent = cur;
				cur = cur->_left;
			} else {
				return make_pair(cur, false);
			}
		}

		// 新建节点
		cur = new Node(data);
		Node* newnode = cur;
		cur->_col = RED;

		// 连接父节点
		if (kot(parent->_data) < kot(data)) {
			parent->_right = cur;
			cur->_parent = parent;
		} else {
			parent->_left = cur;
			cur->_parent = parent;
		}

		// 如果父节点存在且父节点为红色则需要调整
		while (parent && parent->_col == RED) {
			Node* grandfather = parent->_parent;
			if (parent == grandfather->_left) {
				//     g
				//   p   u
				// c 
				// 判断u是否存在和他的颜色
				Node* uncle = grandfather->_right;
				// 如果存在且为红色
				if (uncle && uncle->_col == RED) {
					// 变色
					parent->_col = BLACK;
					uncle->_col = BLACK;
					grandfather->_col = RED;

					// 向上调整
					cur = grandfather;
					parent = cur->_parent;
				} else {
					// 如果不存在或u为黑色，需要判断同侧还是异侧
					// 如果是同侧
					if (cur == parent->_left) {
						//     g
						//   p
						// c
						RotateR(grandfather); // 右旋

						// 调整颜色
						parent->_col = BLACK;
						grandfather->_col = RED;
					} else {
						//    g
						//  p
						//    c
						RotateL(parent);
						RotateR(grandfather);
						cur->_col = BLACK;
						grandfather->_col = RED;
					}
					break;
				}

			} else { // p = g->r
				Node* uncle = grandfather->_left;
				//     g
				//   u   p
				//         c
				// 判断u是否存在和他的颜色
				// 如果存在且为红色
				if (uncle && uncle->_col == RED) {
					// 变色
					parent->_col = BLACK;
					uncle->_col = BLACK;
					grandfather->_col = RED;

					// 向上调整
					cur = grandfather;
					parent = cur->_parent;
				} else {
					if (cur == parent->_right) {
						RotateL(grandfather);
						parent->_col = BLACK;
						grandfather->_col = RED;
					} else {
						//     g
						//   u   p 
						//     c
						//
						RotateR(parent);
						RotateL(grandfather);
						cur->_col = BLACK;
						grandfather->_col = RED;
					}
					break;
				}
			}
		}

		_root->_col = BLACK;

		return make_pair(newnode, true);


	}

	void RotateL(Node* parent) {
		Node* subR = parent->_right;
		Node* subRL = subR->_left;

		parent->_right = subRL;
		subR->_left = parent;

		Node* parentParent = parent->_parent;

		parent->_parent = subR;
		if (subRL)
			subRL->_parent = parent;
		else {
			if (parentParent->_left == parent) {
				parentParent->_left = subR;
			} else {
				parentParent->_right = subR;
			}

			subR->_parent = parentParent;
		}
	}

	void RotateR(Node* parent) {
		Node* subL = parent->_left;
		Node* subLR = subL->_right;

		parent->_left = subLR;
		if (subLR)
			subLR->_parent = parent;

		Node* parentParent = parent->_parent;

		subL->_right = parent;
		parent->_parent = subL;

		if (_root == parent) {
			_root = subL;
			subL->_parent = nullptr;
		} else {
			if (parentParent->_left == parent) {
				parentParent->_left = subL;
			} else {
				parentParent->_right = subL;
			}

			subL->_parent = parentParent;
		}
	}

	void InOrder() {
		_InOrder(_root);
		cout << endl;
	}

	void _InOrder(Node* root) {
		if (root == nullptr)
			return;
		_InOrder(root->_left);
		cout << root->_data << ' ';
		_InOrder(root->_right);
	}

	bool Check(Node* root, int blacknum, const int refVal) {
		if (root == nullptr) {
			if (blacknum != refVal) {
				cout << "存在黑色节点数量不相等的路径" << endl;
				return false;
			}
			return true;
		}

		if (root->_col == RED && root->_parent->_col == RED) {
			cout << "存在连续的红节点" << endl;
			return false;
		}

		if (root->_col == BLACK) {
			++blacknum;
		}
		
		return Check(root->_left, blacknum, refVal) && Check(root->_right, blacknum, refVal);
	}

	bool IsBalance() {
		if (_root == nullptr)
			return true;

		if (_root->_col == RED)
			return false;

		int refVal = 0; // 参考值
		Node* cur = _root;
		while (cur) {
			if (cur->_col == BLACK) {
				++refVal;
			}
			cur = cur->_left;
		}

		int blacknum = 0;
		return Check(_root, blacknum, refVal);
	}

	int Height() {
		return _Height(_root);
	}

	int _Height(Node* root) {
		if (root == nullptr)
			return 0;

		int leftH = _Height(root->_left);
		int rightH = _Height(root->_right);

		return leftH + rightH;
	}

	size_t Size() {
		return _Size(_root);
	}

	size_t _Size(Node* root) {
		if (root == nullptr)
			return 0;
		return _Size(root->_left) + _Size(root->_right) + 1;
	}

	Node* Find(const K& key) {
		Node* cur = _root;
		while (cur) {
			if (cur->_data < key) {
				cur = cur->_right;
			}
			else if (cur->_data > key) {
				cur = cur->_left;
			}
			else {
				return cur;
			}
		}
		return nullptr;
	}

private:
	Node* _root = nullptr;
};

#pragma once
#include"RBTree.h"

namespace xu {
	template<class K, class V>
	class map {
	public:
		struct MapKeyOfT {
			const K& operator() (const pair<K, V>& kv) {
				return kv.first;
			}
		};

		typedef typename RBTree<K, pair<const K, V>, MapKeyOfT>::iterator iterator;
		typedef typename RBTree<K, pair<const K, V>, MapKeyOfT>::const_iterator const_iterator;

		iterator begin() {
			return _t.begin();
		}

		iterator end() {
			return _t.end();
		}

		V& operator[](const K& key) {
			pair<iterator, bool> ret = insert(make_pair(key, V()));
			return ret.first->second;
		}

		pair<iterator, bool> insert(const pair<K, V>& kv) {
			return _t.Insert(kv);
		}
	private:
		RBTree<K, pair<const K, V>, MapKeyOfT> _t;
	};
}

#pragma once
#include"RBTree.h"

namespace xu {
	template<class K>
	class set {
	public:
		struct SetKeyOfT {
			const K& operator(const K& key) {
				return key;
			}
		};

		typedef typename RBTree<K, K, SetKeyOfT>::const_iterator iterator;
		typedef typename RBTree<K, K, SetKeyOfT>::const_iterator const_iterator;

		iterator begin() const {
			return _t.begin();
		}

		iterator end() const {
			return _t.end();
		}

		pair<iterator, bool> insert(const K& key) {
			return _t.Insert(key);
		}
	private:
		RBTree<K, K, SetKeyOfT> _t;
	};
}
```



