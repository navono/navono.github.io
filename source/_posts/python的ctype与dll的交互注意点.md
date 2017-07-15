---
title: python的ctype与dll的交互注意点
date: 2017-01-22 18:29:20
categories: [笔记]
tags: [C++, Python]
---

# 摘要
这篇文章不会告诉读者如何去使用ctypes，如何使用ctypes在网上已经有很多重复、简单的例子。这篇文章只是记录自己在实际开发中，使用python测试一个dll模块中遇到的各种问题以及解决办法。


# 注意点
网络上已经有很多关于ctypes如何与dll交互的文章，大多数也都是点到为止。比如基本类型的使用，指针与引用的处理。但是在实际编程过程中，这里面还有很多细节点需要注意。

- 版本匹配

  这里要求的是安装的python版本位数与交互的dll的编译版本位数要一致。否则在python加载dll时会提示加载失败。

- 数据类型

  很多博客中已经提及了日常所用的一些C的数据类型，比如c_int,c_long。但是如果用C++开发，且同时用了windows类型，COM类型，那么很明显，基本类型是不够的，自己写class又繁琐。例如VARIANT，FILETIME。这些都是已知的类型。那么就需要导入相应的包。windows类型需要相应的wintypes；COM类型可以借助comtypes包。

- 运行环境

  我自己在开发中使用的python3.X，3.X默认的是使用Unicode。如果dll内部接收的是ASCII的字符串，那么就需要先进行转换，再调用API。可以使用create_string_buffer()进行转换。
  如果python加载的dll同时还依赖其他的dll，如果不做任何处理，python加载dll时会提示加载失败。原因是python会从python.exe的目录查找被依赖的dll，结果当然就是找不到，所以失败。解决办法有两个：
  - 第一个是将被依赖的dll的目录加入到环境变量中；
  - 第二个是将被依赖的所有dll拷贝到python.exe目录。
  
  在dll代码中，可能存在参数为const char*类型，调用方传入nullptr或者NULL的情况。此时python运行期会提示错误。原因没有调查，不得而知。变通方案就是将传入的nullptr改成””。
  如果dll的导出API接收const char*类型，而python又需要传入一个空字符串。如果直接使用””，在dll代码中使用param == ‘\0’将会失败。具体原因未知，但是通过调试，dll端接收到的却是是长度为0的字符串。变通方案就是在dll端用再用长度判断一下。
  
  在dll端，我们很大概率会使用结构体存储一个结构。比如下面这种格式：
  ```cpp
  struct Inner{
    int a;
    char *b;
  }
  struct Outter {
  unsigned int count;
  Inner* pInner;
  }
  ```
一个API接收一个Outter数组指针：
```cpp
extern "C" declspec(_dllexport) void func(Outter *param, unsigned int count);
```
此时在python端，我们需要定义相应的数据结构，如果在C的代码中使用了指针，那么对应在python端也需要使用POINTER：

```cpp
class Inner(Structure):
		_fields_ = [("a", c_int),
       			 ("b",c_char_p)]
class Outter(Structure):
		_fields_ = [("count", c_uint),
       			("pInner", POINTER(Inner))]
```
这样，在调用的时候，Outter的实例外层加上byref即可：
```cpp
count = 2
outter_type = Ouuter * count
outter = outter_type()
dll.func(byref(outter), c_uint(count))
```


# matplotlib
发现在测试接口中，有些数据集和时间相关。直接从接口中返回的数据倒是有了，但是很难高效地验证数据的有效性。此时借助matplotlib，将数据集通过图表的形式展示出来，就大大提高了验证数据有效性的效率，同时也很方便观察。