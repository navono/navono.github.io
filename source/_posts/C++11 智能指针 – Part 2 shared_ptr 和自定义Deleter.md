---
title: C++11 智能指针 – Part 2 shared_ptr 和自定义Deleter
categories:
  - 笔记
tags:
  - CPP
  - 翻译
date: 2017-07-19 20:13:34
---

[原文地址](http://thispointer.com/shared_ptr-and-custom-deletor/)

>译注： 本翻译只遵循文章要表达的意图，而不会逐句翻译。

>这个是一系列关于智能指针的文章，谈及的东西都是比较入门的介绍。

[上一篇](http://codingwith.me/2017/07/16/C++11%20%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88%20%E2%80%93%20Part%201%20shared_ptr%20%E6%95%99%E7%A8%8B%E5%92%8C%E7%A4%BA%E4%BE%8B%E4%BB%A3%E7%A0%81/)讲述了`std::shared_ptr`的基本用法。这一篇讲述的是相关的deleter。当`std::shared`对象超出作用域范围时，会自动调用析构函数，当引用计数为0时，会删除相关的裸指针，也就是如何使用自定义的deleter来删除`std::shared`引用的裸指针对象。

通常情况下是直接调用delete。
```cpp
delete pointer;
```

但是当
__`shared_ptr`指向的是数组对象时__，情况就不同了。比如：
```cpp
1
std::shared_ptr<int> p3(new int[12]);
```
此时如果还是使用默认的：
```cpp
delete p3;
```
明显会导致内存泄漏，正确的是应该使用：
```cpp
delete []p3;
```

上述的deleter是在构造``std::shared_ptr`对象时，指定的deleter，通常我们可以自定义，比如：
```cpp
void deleter(Sample * x)
{
	std::cout << "DELETER FUNCTION CALLED\n";
	delete[] x;
}
```
然后再传入`std::shared_ptr`构造函数中：
```cpp
std::shared_ptr<Sample> p3(new Sample[12], deleter);
```

下面是完整的例子：
```cpp
#include <memory>
 
struct Sample
{
	Sample()
	{
		std::cout << "CONSTRUCTOR\n";
	}
	~Sample()
	{
		std::cout << "DESTRUCTOR\n";
	}
};
 
// deleter
void deleter(Sample * x)
{
	std::cout << "DELETER FUNCTION CALLED\n";
	delete[] x;
}
 
int main()
{
	std::shared_ptr<Sample> p3(new Sample[12], deleter);
	return 0;
}
```
输出：
```cpp
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
CONSTRUCTOR
DELETER FUNCTION CALLED
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
DESTRUCTOR
```

作为deleter不仅仅只有上述一种函数的形式，还包括Lambda函数和函数对象。比如：
```cpp
class Deleter
{
	public:
	void operator() (Sample * x) {
		std::cout<<"DELETER FUNCTION CALLED\n";
		delete[] x;
	}
};
 
// 函数对象作为 deleter
std::shared_ptr<Sample> p3(new Sample[12], Deleter());
 
// Lambda函数作为 deleter
std::shared_ptr<Sample> p4(new Sample[12], [](Sample * x){
	std::cout<<"DELETER FUNCTION CALLED\n";
		delete[] x;
});
```

还有一种情况是，在使用资源池的情况下，无需真的释放内存块，而只是将内存或者资源返回给资源池。比如下面这个仿真内存池：
```cpp
#include <memory>
 
struct Sample
{
};
 
// Memory Pool Dummy Kind of Implementation
template<typename T>
class MemoryPool
{
public:
	T * AquireMemory()
	{
		std::cout << "AQUIRING MEMORY\n";
		return (new T());
	}
	void ReleaseMemory(T * ptr)
	{
		std::cout << "RELEASE MEMORY\n";
		delete ptr;
	}
};
int main()
{
	std::shared_ptr<MemoryPool<Sample> > memoryPoolPtr = std::make_shared<
			MemoryPool<Sample> >();
 
	std::shared_ptr<Sample> p3(memoryPoolPtr->AquireMemory(),
			std::bind(&MemoryPool<Sample>::ReleaseMemory, memoryPoolPtr,
					std::placeholders::_1));
	return 0;
}
```