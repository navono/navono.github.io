---
title: C++11 智能指针 – Part 6 unique_ptr
date: 2017-07-29 14:53:03
categories: [笔记]
tags: [CPP, 翻译]
---

[原文地址](http://thispointer.com/c11-unique_ptr-tutorial-and-examples/)

>译注： 本翻译只遵循文章要表达的意图，而不会逐句翻译。

>这个是一系列关于智能指针的文章，谈及的东西都是比较入门的介绍。

# 什么是`std::unique_ptr`？
智能指针出了`std::shared_ptr`，还包括另外一种`std::unique_ptr`。`std::unique_ptr`与`std::shared_ptr`功能一样，除了`std::unique_ptr`是独占资源所有权。意思是`std::unique_ptr`对象不能与其他的`std::unique_ptr`对象共享资源。`std::unique_ptr`也接受一个裸指针来创建对象。
```cpp
#include <iostream>
#include <memory>
 
struct Task
{
	int mId;
	Task(int id ) :mId(id)
	{
		std::cout<<"Task::Constructor"<<std::endl;
	}
	~Task()
	{
		std::cout<<"Task::Destructor"<<std::endl;
	}
};
 
int main()
{
	// Create a unique_ptr object through raw pointer
	std::unique_ptr<Task> taskPtr(new Task(23));
 
	//Access the element through unique_ptr
	int id = taskPtr->mId;
 
	std::cout<<id<<std::endl;
 
 
	return 0;
}
```
输出：
```cpp
Task::Constructor
23
Task::Destructor
```

`std::unique_ptr`对象`taskPtr`接收一个裸指针作为参数。当函数退出时，`taskPtr`对象会超出作用域，析构函数会被调用。在析构函数中，`taskPtr`会删除与之相关的裸指针。

不管函数是正常退出还是异常退出，`taskPtr`的析构函数都会被调用。因此不会造成内存泄漏的情况。


# unique指针的独享所有权

独享所有权的意思是无法拷贝`std::unique_ptr`对象。只能`move`。每一个`std::unique_ptr`对象都只是裸指针的唯一拥有者，所有`std::unique_ptr`对象内部无需管理引用计数。

# 创建一个空的`std::unique_ptr`对象

```cpp
// 空的`std::unique_ptr`对象
std::unique_ptr<int> ptr1;
```
因为没有裸指针与之关联，所以`ptr1`是空的。

# 检查`std::unique_ptr`对象是否为空
方法1：
```cpp
if(!ptr1)
	std::cout<<"ptr1 is empty"<<std::endl;
```

方法2：
```cpp
if(ptr1 == nullptr)
	std::cout<<"ptr1 is empty"<<std::endl;
```

# 使用裸指针创建`std::unique_ptr`对象
```cpp
std::unique_ptr<Task> taskPtr(new Task(23));
```
不能通过赋值的方式来创建`std::unique_ptr`对象。
```cpp
std::unique_ptr<Task> taskPtr2 = new Task(); // Compile Error
```

# 重置`std::unique_ptr`对象
调用`reset()`函数会导致`std::unique_ptr`对象被重置，也就是与之关联的裸指针会被删除，同时`std::unique_ptr`对象会被置空。
```cpp
taskPtr.reset();
```

# `std::unique_ptr`对象无法被拷贝
```cpp
// 创建 `unique_ptr` 对象
std::unique_ptr<Task> taskPtr2(new Task(55));

// 编译错误
std::unique_ptr<Task> taskPtr3 = taskPtr2;

// 编译错误
taskPtr = taskPtr2;
```

# 转移`std::unique_ptr`对象所有权
无法拷贝`std::unique_ptr`对象，但是可以转移`std::unique_ptr`对象的所有权。
```cpp
std::unique_ptr<Task> taskPtr2(new Task(55));

std::unique_ptr<Task> taskPtr4 = std::move(taskPtr2);
 
if(taskPtr2 == nullptr)
	std::cout<<"taskPtr2 is  empty"<<std::endl;

// `taskPtr2`的所有权被转移到 `taskPtr4`对象
if(taskPtr4 != nullptr)
	std::cout<<"taskPtr4 is not empty"<<std::endl;

std::cout<<taskPtr4->mId<<std::endl;
```
`taskPtr2`对象在转移所有权后会被置为空。

# 释放裸指针
通过调用`release()`函数，可以从`std::unique_ptr`对象中将裸指针释放出来，它返回一个裸指针。
```cpp
std::unique_ptr<Task> taskPtr5(new Task(55));
 
if(taskPtr5 != nullptr)
	std::cout<<"taskPtr5 is not empty"<<std::endl;
 
Task * ptr = taskPtr5.release();
 
if(taskPtr5 == nullptr)
	std::cout<<"taskPtr5 is empty"<<std::endl;
```

完整例子：
```cpp
#include <iostream>
#include <memory>

struct Task
{
	int mId;
	Task(int id ) :mId(id)
	{
		std::cout<<"Task::Constructor"<<std::endl;
	}
	~Task()
	{
		std::cout<<"Task::Destructor"<<std::endl;
	}
};

int main()
{
	// Empty unique_ptr object
	std::unique_ptr<int> ptr1;

	// Check if unique pointer object is empty
	if(!ptr1)
		std::cout<<"ptr1 is empty"<<std::endl;

	// Check if unique pointer object is empty
	if(ptr1 == nullptr)
		std::cout<<"ptr1 is empty"<<std::endl;

	// can not create unique_ptr object by initializing through assignment
	// std::unique_ptr<Task> taskPtr2 = new Task(); // Compile Error

	// Create a unique_ptr object through raw pointer
	std::unique_ptr<Task> taskPtr(new Task(23));

	// Check if taskPtr is empty or it has an associated raw pointer
	if(taskPtr != nullptr)
		std::cout<<"taskPtr is  not empty"<<std::endl;

	//Access the element through unique_ptr
	std::cout<<taskPtr->mId<<std::endl;

	std::cout<<"Reset the taskPtr"<<std::endl;
	// Reseting the unique_ptr will delete the associated
	// raw pointer and make unique_ptr object empty
	taskPtr.reset();

	// Check if taskPtr is empty or it has an associated raw pointer
	if(taskPtr == nullptr)
		std::cout<<"taskPtr is  empty"<<std::endl;


	// Create a unique_ptr object through raw pointer
	std::unique_ptr<Task> taskPtr2(new Task(55));

	if(taskPtr2 != nullptr)
		std::cout<<"taskPtr2 is  not empty"<<std::endl;

	// unique_ptr object is Not copyable
	//taskPtr = taskPtr2; //compile error

	// unique_ptr object is Not copyable
	//std::unique_ptr<Task> taskPtr3 = taskPtr2;

	{
		// Transfer the ownership

		std::unique_ptr<Task> taskPtr4 = std::move(taskPtr2);


		if(taskPtr2 == nullptr)
			std::cout<<"taskPtr2 is  empty"<<std::endl;

		// ownership of taskPtr2 is transfered to taskPtr4
		if(taskPtr4 != nullptr)
			std::cout<<"taskPtr4 is not empty"<<std::endl;

		std::cout<<taskPtr4->mId<<std::endl;

		//taskPtr4 goes out of scope and deletes the assocaited raw pointer
	}

	// Create a unique_ptr object through raw pointer
	std::unique_ptr<Task> taskPtr5(new Task(55));

	if(taskPtr5 != nullptr)
		std::cout<<"taskPtr5 is not empty"<<std::endl;

	// Release the ownership of object from raw pointer
	Task * ptr = taskPtr5.release();

	if(taskPtr5 == nullptr)
		std::cout<<"taskPtr5 is empty"<<std::endl;

	std::cout<<ptr->mId<<std::endl;

	delete ptr;

	return 0;
}
```

输出：

```cpp
ptr1 is empty
ptr1 is empty
Task::Constructor
taskPtr is  not empty
23
Reset the taskPtr
Task::Destructor
taskPtr is  empty
Task::Constructor
taskPtr2 is  not empty
taskPtr2 is  empty
taskPtr4 is not empty
55
Task::Destructor
Task::Constructor
taskPtr5 is not empty
taskPtr5 is empty
55
Task::Destructor
```
