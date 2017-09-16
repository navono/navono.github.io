---
title: 如何读取OLE控件的PropertyBag
date: 2017-09-16 08:30:32
categories: [笔记]
tags: [CPP, OLE]
---

# 目的
&ensp;&ensp;&ensp;本篇的目的就是记录下如何获取OLE控件的属性页的全部属性。想要达到这个目的的时候，在网上搜寻了一番，基本没找到可用的信息。因此就开始自我探寻。碰巧之前在开发OLE的时候，见到有个叫TstCon.exe的工具，这个工具的功能就是类似加载系统注册的OCX控件。后来在[github](https://github.com/Microsoft/VCSamples)上找到了源码，因此问题也就迎刃而解了。在此简单记录一下。

# 实现方法

## 第一步
&ensp;&ensp;&ensp;获取控件的接口`IPersistPropertyBag`指针，这个可以通过`QueryInterface`取得：
```cpp
HRESULT hResult;
IPersistPropertyBagPtr pPersistPropertyBag;

hResult = m_lpObject->QueryInterface( IID_IPersistPropertyBag,(void**)&pPersistPropertyBag );
```

## 第二步
&ensp;&ensp;&ensp;在`IPersistPropertyBagPtr`接口中，还没办法直接获取PropertyBag。而是调其方法，将属性保存到一个实现了接口`IPropertyBag`的类中：
```cpp
IPropertyBag* pPropertyBag
hResult = pPersistPropertyBag->Save(pPropertyBag, TRUE, TRUE);
```

## 第三步
&ensp;&ensp;&ensp;实现一个继承了`IPropertyBag`的类。`IPropertyBag`接口有以下方法需要实现：
```cpp
STDMETHOD(Read)(LPCOLESTR pszPropName, VARIANT* pvarValue, IErrorLog* pErrorLog);
STDMETHOD(Write)(LPCOLESTR pszPropName, VARIANT* pvarValue);
```

也就是一个读一个写。写方法由`pPersistPropertyBag->Save`将OCX控件内的属性读入到`IPropertyBag`的实现类中。然后可以用读方法，通过属性名获取其值。通常情况下可能还需要实现`IUnknown`的三个接口用来管理COM指针：
```cpp
STDMETHOD_(ULONG, AddRef)();
STDMETHOD_(ULONG, Release)();
STDMETHOD(QueryInterface)(REFIID iid, void** ppInterface);
```

&ensp;&ensp;&ensp;可以确定的是，在`IPropertyBag`的实现类中需要用队列或者某种方式保存KV对。这样才能在`pPersistPropertyBag->Save`调用之后，属性值能被外界读取。而在`TstCon`的源码中恰好实现了一个PropertyBag类。几乎可以重用它的所有代码。

# 总结
&ensp;&ensp;&ensp;`TstCon`的源码中有很多设计是比较经典的，因为它实现了一个`OCX控件`容器的功能。如果是在基于`MFC`的`OCX控件`的展示，配置等，有很多代码的设计可以借鉴。当然这个技术实在是太古老了，目前在新项目中很少人会选择这样的技术，网上的相关资源也越来越少，遇到棘手问题只能自行解决。了解这个技术所带来的回报也越来越低，能用到的最多也只是维护老项目过程中。