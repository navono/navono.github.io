---
title: C++11 智能指针 – Part 3 shared_ptr vs 指针
date: 2017-07-22 19:09:31
categories: [笔记]
tags: [CPP, 翻译]
---

[原文地址](http://thispointer.com/how-shared_ptr-object-is-different-from-a-raw-pointer/)

>译注： 本翻译只遵循文章要表达的意图，而不会逐句翻译。

>这个是一系列关于智能指针的文章，谈及的东西都是比较入门的介绍。


我们从以下几个方面对`shared_ptr`对象和裸指针进行比较。

# 缺失 ++，-- 和 [] 操作符
和裸指针相比，`shared_ptr`对象只提供以下操作符：
- ->, *
- 比较操作符

不提供以下：
- 算数运算，比如：+，-，++，--
- []操作符

看下代码：
```cpp
#include<iostream>
#include<memory>

struct Sample
{
	void dummyFunction()
	{
		std::cout << "dummyFunction" << std::endl;
	}
};

int main()
{
	std::shared_ptr<Sample> ptr = std::make_shared<Sample>();

	(*ptr).dummyFunction(); // Will Work

	ptr->dummyFunction(); // Will Work

	// ptr[0]->dummyFunction(); // 编译会失败。
	// ptr++;  // 编译会失败。
	//ptr--;  // 编译会失败。

	std::shared_ptr<Sample> ptr2(ptr);

	if (ptr == ptr2) // 能运行
		std::cout << "ptr and ptr2 are equal" << std::endl;

	return 0;
}
```

输出：
```cpp
dummyFunction
dummyFunction
ptr and ptr2 are equal
```

# NULL 检查

创建一个`shared_ptr`对象而不对其赋值，那么它就是空的。然而，如果我们创建一个裸指针而不对其赋值，那么它有可能是指向一个垃圾内存地址，同时我们又没办法去验证它所指的内存地址是垃圾地址还是有效地址。

我们可以对`shared_ptr`进行这样的检测：
```cpp
std::shared_ptr<Sample> ptr3;
if(!ptr3)
	std::cout<<"Yes, ptr3 is empty" << std::endl;
if(ptr3 == NULL)
	std::cout<<"ptr3 is empty" << std::endl;
if(ptr3 == nullptr)
	std::cout<<"ptr3 is empty" << std::endl;
```

也可以访问`shared_ptr`内部的裸指针：
```cpp
std::shared_ptr<Sample> ptr = std::make_shared<Sample>();
Sample * rawptr = ptr.get();
```

但是通常不建议这么做。因为如果将内部的指针取出后进行删除，当`shared_ptr`超出作用域同时引用计数为0时再次删除内部的资源时会导致程序崩溃。