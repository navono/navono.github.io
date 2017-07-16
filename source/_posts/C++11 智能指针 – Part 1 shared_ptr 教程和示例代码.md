---
title: C++11 智能指针 – Part 1 shared_ptr 教程和示例代码
date: 2017-07-16 15:42:26
categories: [笔记]
tags: [CPP, 翻译]
---

[原文地址](http://thispointer.com/learning-shared_ptr-part-1-usage-details/)

>译注： 本翻译只遵循文章要表达的意图，而不会逐句翻译。

>这个是一系列关于智能指针的文章，谈及的东西都是比较入门的介绍。


# 什么是std::shared_ptr<>
`shared_ptr`是`C++11`提供的智能指针的一种。用来在离开作用域的时候，自动删除相关联指针的技术。因此它可以帮助我们处理一些因为疏忽大意而导致的内存泄漏和指针悬挂问题。


# shared_ptr和共享所有权
谈及`shared_ptr`就不得不谈及资源所有权的问题。不同的`shared_ptr`可以关联到t相同的资源指针，内部使用引用计数来进行管理。

__每个`shared_ptr`对象内部都指向两个内存地址：__
- 指向资源对象
- 指向用于引用计数的数据结构

__共享所有权是如何通过引用计数来实现的？__
- 当一个新的`shared_ptr`对象和一个裸指针关联时，在构造函数内会对此裸指针关联的引用计数加1
- 当`shared_ptr`超出作用域时，在其析构函数内，会对其关联的引用计数减1。如果引用计数变为0,也就意味着没有其他的`shared_ptr`对象和裸指针所指向的内存相关联了，那么就是使用“delete”函数来释放内存空间。


# 创建一个`shared_ptr`对象
## 方法1：使用构造
```cpp
std::shared_ptr<int> p1(new int());
```
上述代码在堆上分配了两块内存：
1. 对象int
2. 管理`shared_ptr`对象的引用计数的内存。初始值为1.


__检查一个`shared_ptr`对象的引用计数：__
```cpp
p1.use_count();
```

## 方法2：使用std::make_shared<T>
不能直接将指针赋值给一个`shared_ptr`对象
```cpp
//Compile Error
std::shared_ptr<int> p1 = new int(); // Compile error
```

因为`shared_ptr`的构造函数式显示接收一个参数,想方法1一样。创建`shared_ptr`对象最好的方式是通过`std::make_shared`。
```cpp
std::shared_ptr<int> p1 = std::make_shared<int>();
```

好处是`std::make_ptr`只会分配一个内存块,也就是 new operator只会调用一次。

## 从裸指针分离
使用`reset()`函数会使`shared_ptr`对象的引用计数减1。
```cpp
p1.reset();
```

`reset()`还有一种用法，就是指向一个新的指针，内部的引用计数会被重置为1。
```cpp
p1.reset(new int(34));
```

从裸指针分离的第二个方法是直接赋值为`nullptr`。
```cpp
p1 = nullptr;
```

__`shared_ptr`是一个*伪*指针（psuedo pointer）__
`shared_ptr`会假装成一个正常的指针。比如我们可用 `*`和`->`来操作`shared_ptr`对象。


完整的示例如下：
```cpp
#include <iostream>
#include  <memory> // shared_ptr的t头文件
 
int main()
{
  // 通过`make_shared`创建一个`shared_ptr`对象
	std::shared_ptr<int> p1 = std::make_shared<int>();
	*p1 = 78;
	std::cout << "p1 = " << *p1 << std::endl;
 
  // 打印引用计数
	std::cout << "p1 Reference count = " << p1.use_count() << std::endl;
 
  // 第二个`shared_ptr`对象内部会指向同一个指针，因此引用计数会变成2
	std::shared_ptr<int> p2(p1);
 
  // 打印引用计数
	std::cout << "p2 Reference count = " << p2.use_count() << std::endl;
	std::cout << "p1 Reference count = " << p1.use_count() << std::endl;
 
  // `shared_ptr`对象比较
	if (p1 == p2)
	{
		std::cout << "p1 and p2 are pointing to same pointer\n";
	}
 
	std::cout<<"Reset p1 "<<std::endl;
 
	p1.reset();
 
  // 重置 `shared_ptr`对象，此对象内部不再指向裸指针，因此引用计数会变成0

	std::cout << "p1 Reference Count = " << p1.use_count() << std::endl;
 
  // 重置 `shared_ptr`对象，此对象内部指向一个新的裸指针，因此引用计数会变成1
 
	p1.reset(new int(11));
 
	std::cout << "p1  Reference Count = " << p1.use_count() << std::endl;
 
  // 使用nullptr进行赋值
	p1 = nullptr;
 
	std::cout << "p1  Reference Count = " << p1.use_count() << std::endl;
 
	if (!p1)
	{
		std::cout << "p1 is NULL" << std::endl;
	}
	return 0;
}
```

输出：
```cpp
p1 = 78
p1 Reference count = 1
p2 Reference count = 2
p1 Reference count = 2
p1 and p2 are pointing to same pointer
Reset p1 
p1 Reference Count = 0
p1  Reference Count = 1
p1  Reference Count = 0
p1 is NULL
```