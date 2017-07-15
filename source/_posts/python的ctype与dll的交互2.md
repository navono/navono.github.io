---
title: python的ctype与dll的交互2
date: 2017-03-12 18:29:20
categories: [笔记]
tags: [CPP, Python]
---

# 摘要
[上一篇](http://codingeek.me/2017/01/22/python%E7%9A%84ctype%E4%B8%8Edll%E7%9A%84%E4%BA%A4%E4%BA%92/)文章说了python与dll交互间的数据结构问题。这篇文章说下dll中的回调问题。介绍python的c回调的文章网上有很多，但很多都是无法工作。因此在此记录下整个过程。

# C的回调
通常c写的回调都是类似这样的结构：

```cpp
struct A_s
{
	const char* a1;
	unsigned int a2;
};
typedef struct A_s A_t;
TESTDLL_API int function1(void (*outputcallback)(const A_t* a, void* b), void* param);
```

在c或者C++中调用的话，第一个参数可以传入一个签名匹配到回调的方法或者lambda对象（Lambda参考[这篇](http://codingeek.me/2017/03/04/Callback-in-C-and-C/)）。



# python的回调
在python中，需要定义相应的数据结构，接下来就是回调的原型定义。

```python
class A(Structure):
    _fields_ = [
        ("a1", c_char_p),
        ("a2", c_int)]
CMPFUNC = CFUNCTYPE(None, POINTER(A), c_void_p)
```

CFUNCTYPE的第一个参数是回调的返回值，接下是回调的参数。有一点需要注意的是，任何传入c接口的参数都是ctypes类型，而不是python的内置内类。

```python
self.dllModule.function1(self.cb, c_int(10))
```

回调中的第一个参数是一个对象，这个对象包含了一个contents字段，在这个字段中才是我们自定义的字段。

```python
def callback(self, a, b):
        print(dir(a))
        print(a.contents.a1, a.contents.a2)
        print(b)
```

# 源码
源码在[这里](https://github.com/navono/blog_code/tree/master/python-callback)