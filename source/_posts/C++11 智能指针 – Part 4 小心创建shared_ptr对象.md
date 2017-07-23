---
title: C++11 智能指针 – Part 4 小心创建shared_ptr对象
date: 2017-07-23 15:12:11
categories: [笔记]
tags: [CPP, 翻译]
---

[原文地址](http://thispointer.com/create-shared_ptr-objects-carefully/)

>译注： 本翻译只遵循文章要表达的意图，而不会逐句翻译。

>这个是一系列关于智能指针的文章，谈及的东西都是比较入门的介绍。

在创建`shared_ptr`对象时应该要注意两点：

1. 不要使用裸指针来创建两个以上的`shared_ptr`对象
2. 不要从栈内存上创建`shared_ptr`对象

# 不要使用裸指针来创建两个以上的`shared_ptr`对象
因为每个`shared_ptr`对象是不清楚其他的`shared_ptr`对象与其共享了裸指针对象。
```cpp
int * rawPtr = new int();
std::shared_ptr<int> ptr_1(rawPtr);
std::shared_ptr<int> ptr_2(rawPtr);
```
当其中之一超过作用域范围，那么就会删除与其相关的裸指针对象。这样一来就导致了另外的`shared_ptr`对象指向的裸指针变成了悬挂指针（dangling pointer），当期超出作用域时，再次删除内部裸指针就会导致程序崩溃。
```cpp
#include<iostream>
#include<memory>
typedef struct Sample {
Sample() {
    internalValue = 0;
    std::cout<<"Constructor"<<std::endl;
}
~Sample() {
    std::cout<<"Destructor"<<std::endl;
}
}Sample;
int main()
{
    {
    Sample * rawPtr = new Sample();
    std::shared_ptr<Sample> ptr_1(rawPtr);
 
        {
        std::shared_ptr<Sample> ptr_2(rawPtr);
        }
// 因为`ptr_2`不知道相同的裸指针也被用在了其他的`shared_ptr`对象上，因此当`ptr_2`超出作用域后，会将其相关联的裸指针删除。
 
// 此时，`ptr_1`内部关联的是一个悬挂指针。因此当其超出作用域后会试图去删除一个已被删掉的裸指针，程序崩溃。
    }
return 0;
}
```

# 不要从栈内存上创建`shared_ptr`对象
看下下面的代码：
```cpp
#include<iostream>
#include<memory>

int main()
{
int x = 12;
std::shared_ptr<int> ptr(&x);
return 0;
}
```

`shared_ptr`对象所预期的内存是从堆上开辟的，因此在其超出作用域后，如果引用计数为0就会从堆上删除。而上面的代码中可以看到，`shared_ptr`对象相关联的内存是在栈上，因此当其超出作用域试图删除相关的内存时，会导致程序崩溃。

正确创建`shared_ptr`对象不是从裸指针，而是使用`make_shared<>`。
```cpp
std::shared_ptr<int> ptr_1 = make_shared<int>();

std::shared_ptr<int> ptr_2 (ptr_1);
```