---
title: Callback in C and C++
date: 2017-03-04 18:29:20
categories: [笔记]
tags: [C++]
---

# 动机
前面一段时间一直在看[g3log](https://github.com/KjellKod/g3log)和[libuv](https://github.com/libuv/libuv)的源码。在看完也写了点例子操作后，看到g3log里面只有[FileSink](https://github.com/navono/g3Log_Sinks)，因此想自己写一个SocketSink，这个版本实现只是功能上的实现。


# 问题
因为g3log是C++11开发的，但是libuv库又是一个C库，而且是大量使用回调的模式的事件驱动库。因此两者的结合看起来很奇怪。站在使用者角度，当然是使用C++11风格的好。但是在试过以下代码，发现C++11的Lambda好像和C的回调不是那么匹配。

```cpp
auto connect_cb = [](uv_connect_t* req, int status) {
	uv_read_start((uv_stream_t*)req->handle, on_alloc, on_read);
};
    
uv_tcp_connect(&con_req, client, (const sockaddr*)&addr, connect_cb);
```

在VS2010中编译不通过，原因是因为Lambda在生成Closure的时候使用了上下文，而C的回调刚好和调用的上下文无关的。因此无法将function形式的函数转换成函数指针。但是在VS2015中则可以。

但是又存在另一个问题，就是如果想要将on_alloc和on_read也使用Lambda就会存在问题，因为在connect_cb内部使用的话：

```cpp
auto alloc_cb = [](uv_handle_t* handle, size_t suggested_size, uv_buf_t* buf) {
    buf->base = (char*)malloc(suggested_size);
    buf->len = suggested_size;
};
auto connect_cb = [](uv_connect_t* req, int status) {
    uv_read_start((uv_stream_t*)req->handle, alloc_cb, on_read);
};
uv_tcp_connect(&con_req, client, (const sockaddr*)&addr, connect_cb);
```

就需要capture为引用或者拷贝，这样就导致了在使用*connect_cb*会报出类似：
>error C3493: ‘alloc_cb’ cannot be implicitly captured because no default capture mode has been specified 

>error C2664: ‘int uv_tcp_connect(uv_connect_t ,uv_tcp_t ,const sockaddr *,uv_connect_cb)’: cannot convert argument 4 from ‘int’ to ‘uv_connect_cb’

这样的错误。但是在connect_cb中使用[&] capture，又会导致

>error C2664: ‘int uv_tcp_connect(uv_connect_t ,uv_tcp_t ,const sockaddr *,uv_connect_cb)’: cannot convert argument 4 from ‘run_tcp_client::‘ to ‘uv_connect_cb’

陷入了两难境地。google了一把，就想将一些东西记录在这。


# 回调
## 传统方案
大部分想到的是传统方案，也就是使用接口类。
```cpp
//------------------------------------------------------------------------
// Abstract Base Class
// Those who want to provide a callback must derive from this class and
// provide an implementation of cbiCallbackFunction().
class CallbackInterface
{
public:
    // The prefix "cbi" is to prevent naming clashes.
    virtual int cbiCallbackFunction(int) = 0;
};
//------------------------------------------------------------------------
// "Caller" allows a callback to be connected.  It will call that callback.
class Caller
{
public:
    // Clients can connect their callback with this
    void connectCallback(CallbackInterface *cb)
    {
        m_cb = cb;
    }
    // Test the callback to make sure it works.
    void test()
    {
        printf("Caller::test() calling callback...\n");
        int i = m_cb -> cbiCallbackFunction(10);
        printf("Result (20): %d\n", i);
    }
private:
    // The callback provided by the client via connectCallback().
    CallbackInterface *m_cb;
};
//------------------------------------------------------------------------
// "Callee" can provide a callback to Caller.
class Callee : public CallbackInterface
{
public:
    // The callback function that Caller will call.
    int cbiCallbackFunction(int i)  
    { 
        printf("  Callee::cbiCallbackFunction() inside callback\n");
        return 2 * i; 
    }
};
//------------------------------------------------------------------------
```

使用方法：
```cpp
Caller caller;
Callee callee;
// Connect the callback
caller.connectCallback(&callee);
// Test the callback
caller.test();
```

__优点：__
- 如果要处理多个回调，可以直接往接口类里添加
- 清晰简单

__缺点：__
- 接口可能存在名字冲突
- 只能继承一次


## 函数指针
```cpp
//------------------------------------------------------------------------
// Callback function pointer.
typedef int(*CallbackFunctionPtr)(void*, int);
//------------------------------------------------------------------------
// "Caller" allows a callback to be connected.  It will call that callback.
class Caller
{
public:
    // Clients can connect their callback with this.  They can provide
    // an extra pointer value which will be included when they are called.
    void connectCallback(CallbackFunctionPtr cb, void *p)
    {
        m_cb = cb;
        m_p = p;
    }
    // Test the callback to make sure it works.
    void test()
    {
        printf("Caller::test() calling callback...\n");
        int i = m_cb(m_p, 10);
        printf("Result (20): %d\n", i);
    }
private:
    // The callback provided by the client via connectCallback().
    CallbackFunctionPtr m_cb;
    // The additional pointer they provided (it's "this").
    void *m_p;
};
//------------------------------------------------------------------------
// "Callee" can provide a callback to Caller.
class Callee
{
public:
    // This static function is the real callback function.  It's compatible
    // with the C-style CallbackFunctionPtr.  The extra void* is used to
    // get back into the real object of this class type.
    static int staticCallbackFunction(void *p, int i)
    {
        // Get back into the class by treating p as the "this" pointer.
        ((Callee *)p) -> callbackFunction(i);
    }
    // The callback function that Caller will call via 
    // staticCallbackFunction() above.
    int callbackFunction(int i)  
    {
        printf("  Inside callback\n");
        return 2 * i; 
    }
};
```

使用方法：

```cpp
Caller caller;
Callee callee;
// Connect the callback.  Send the "this" pointer for callee as the 
// void* parameter.
caller.connectCallback(Callee::staticCallbackFunction, &callee);
// Test the callback
caller.test();
```

__优点：__
- 和C风格的回调兼容

__缺点：__
- 实现优点复杂，优点绕


## C++11 Lambda
```cpp
class Callee
{
public:
    Callee(int i) : m_i(i) { }
    // The callback function that Caller will call.
    int callbackFunction(int i)
    {
        printf("  Callee::callbackFunction() inside callback\n");
        return m_i * i; 
    }
private:
    // To prove "this" is indeed valid within callbackFunction().
    int m_i;
};
//------------------------------------------------------------------------
typedef std::function<int(int)> CallbackFunction;
class Caller
{
public:
    // Clients can connect their callback with this.
    void connectCallback(CallbackFunction cb)
    {
        m_cb = cb;
    }
    // Test the callback to make sure it works.
    void test()
    {
        printf("Caller::test() calling callback...\n");
        int i = m_cb(10);
        printf("Result (50): %d\n", i);
    }
private:
    // The callback provided by the client via connectCallback().
    CallbackFunction m_cb;
};
```

使用方法：

```cpp
Caller caller;
Callee callee(5);
// Connect the callback.  Like with the C-style function pointer and 
// static function, we use a lambda to get back into the object.
caller.connectCallback(
    [&callee](int i) { return callee.callbackFunction(i); });
// Test the callback
caller.test();
```

## 模板函子
实现[在这](http://www.tutok.sk/fastgl/callback.html)。可以用C++11的可变模板实现。


# 解决方案
最终要解决上述的问题是，要解决capture的问题。解决方案是引入一个“bounce”函数。解决上下文问题。

```cpp
static auto/*static std::function<void(uv_connect_t* req, int status)>*/ connect_bounce
		= [&](uv_connect_t* req, int status) {
		uv_read_start((uv_stream_t*)req->handle, alloc_cb, read_cb);
	};
auto connect_cb = [](uv_connect_t* req, int status) {
    connect_bounce(req, status);
};
```

这样，alloc_cb和read_cb就可以使用正常的lambda了。

# 引用
- http://tedfelix.com/software/c++-callbacks.html
- http://www.alecjacobson.com/weblog/?p=3779
- https://www.cabbages-and-kings.net/2014/08/11/c_style_callbacks_with_c_functions.html