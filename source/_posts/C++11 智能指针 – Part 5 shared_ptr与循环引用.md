---
title: C++11 智能指针 – Part 5 shared_ptr与循环引用
date: 2017-07-26 19:04:19
categories: [笔记]
tags: [CPP, 翻译]
---

[原文地址](http://thispointer.com/shared_ptr-binary-trees-and-the-problem-of-cyclic-references/)

>译注： 本翻译只遵循文章要表达的意图，而不会逐句翻译。

>这个是一系列关于智能指针的文章，谈及的东西都是比较入门的介绍。

使用智能指针的主要优点是能够在合适的时间来释放相关的内存。但是使用不当也会将这个主要优点变成缺点。

假设我们要实现一个二叉树，用左右两个指针来保存孩子节点。
```cpp
#include <memory>
Node
{
    int value;
    public:
    std::shared_ptr<Node> leftPtr;
    std::shared_ptr<Node> rightPtr;
    Node(int val) : value(val) {
         std::cout<<"Contructor"<<std::endl;
    }
    ~Node() {
         std::cout<<"Destructor"<<std::endl;
    }
};
```
然后可以这么使用
```cpp
#include <iostream>
int main()
{
    std::shared_ptr<Node> ptr = std::make_shared<Node>(4);
    ptr->leftPtr = std::make_shared<Node>(2);
    ptr->rightPtr = std::make_shared<Node>(5);
    return 0;
}
```
运行后在控制台会输出三次构造和三次析构。完全按照我们想要的执行。
![正常](normal.PNG)

但是此时如果想要增加一个小需求，也就是每个节点保存父节点。我们可能会想到这样做：
```cpp
#include <memory>
class Node
{
    int value;
    public:
    std::shared_ptr<Node> leftPtr;
    std::shared_ptr<Node> rightPtr;
    std::shared_ptr<Node> parentPtr;
    Node(int val) : value(val)     {
         std::cout<<"Contructor"<<std::endl;
    }
    ~Node()     {
         std::cout<<"Destructor"<<std::endl;
    }
};
```
然后这样使用：
```cpp
#include <iostream>
int main()
{
    std::shared_ptr<Node> ptr = std::make_shared<Node>(4);
    ptr->leftPtr = std::make_shared<Node>(2);
    ptr->leftPtr->parentPtr = ptr;
    ptr->rightPtr = std::make_shared<Node>(5);
    ptr->rightPtr->parentPtr = ptr;
    std::cout<<"ptr reference count = "<<ptr.use_count()<<std::endl;
    std::cout<<"ptr->leftPtr reference count = "<<ptr->leftPtr.use_count()<<std::endl;
    std::cout<<"ptr->rightPtr reference count = "<<ptr->rightPtr.use_count()<<std::endl;
    return 0;
}
```
但是此时在控制台输出中，只看到了三次构造，没有析构。
![内存泄漏](leak.PNG)

原因是这里引入了循环依赖。`shared_ptr`对象释放内存的条件是内部引用计数为0。上述的例子中，`ptr`对象的内部引用计数不可能为0。分析一下。

当`ptr`析构被调用：
- 引用计数减1
- 检查引用计数的值是否为0，但实际上为2，因为被左右子节点引用了
- 左右子节点只有在`ptr`对象被正确删除后才会被删除，但是此时`ptr`的引用计数一直大于0，所以不会被删除
- 最终，`ptr`和子节点的内存都没有被删除，因此没有析构被调用

解决此问题的办法是使用`weak_ptr`。`weak_ptr`只能对资源共享，也就是会增加内部的引用计数，但是不会对实际资源进行引用。因此`weak_ptr`对象不能使用操作符`*`和`->`来访问相关的资源。只能通过调用`lock()`函数来创建`shared_ptr`对象来操作。
```cpp
#include <iostream>
#include <memory>
int main()
{
    std::shared_ptr<int> ptr = std::make_shared<int>(4);
    std::weak_ptr<int> weakPtr(ptr);
    std::shared_ptr<int> ptr_2 =  weakPtr.lock();
    if(ptr_2)
        std::cout<<(*ptr_2)<<std::endl;
    std::cout<<"Reference Count :: "<<ptr_2.use_count()<<std::endl;   
    if(weakPtr.expired() == false)
        std::cout<<"Not expired yet"<<std::endl;   
    return 0;
}
```
输出：
![weak_ptr](weak_ptr.PNG)

这里有个需要注意的是，`lock`函数可能返回空的`shared_ptr`对象。

将我们的`Node`的`parentPtr`类型修改成`weak_ptr`即可解决循环引用的问题。
```cpp
#include <iostream>
#include <memory>
class Node
{
    int value;
    public:
    std::shared_ptr<Node> leftPtr;
    std::shared_ptr<Node> rightPtr;
    // Just Changed the shared_ptr to weak_ptr
    std::weak_ptr<Node> parentPtr;
    Node(int val) : value(val)     {
         std::cout<<"Contructor"<<std::endl;
    }
    ~Node()     {
         std::cout<<"Destructor"<<std::endl;
    }
};
int main()
{
    std::shared_ptr<Node> ptr = std::make_shared<Node>(4);
    ptr->leftPtr = std::make_shared<Node>(2);
    ptr->leftPtr->parentPtr = ptr;
    ptr->rightPtr = std::make_shared<Node>(5);
    ptr->rightPtr->parentPtr = ptr;
    std::cout<<"ptr reference count = "<<ptr.use_count()<<std::endl;
    std::cout<<"ptr->leftPtr reference count = "<<ptr->leftPtr.use_count()<<std::endl;
    std::cout<<"ptr->rightPtr reference count = "<<ptr->rightPtr.use_count()<<std::endl;
    std::cout<<"ptr->rightPtr->parentPtr reference count = "<<ptr->rightPtr->parentPtr.lock().use_count()<<std::endl;
    std::cout<<"ptr->leftPtr->parentPtr reference count = "<<ptr->leftPtr->parentPtr.lock().use_count()<<std::endl;
    return 0;
}
```
输出：
![weak_ptr](weak_ptr2.PNG)