---
title: JavaScript是如何工作的：引擎、运行时与调用栈
date: 2018-02-03 11:08:53
categories: [笔记]
tags: [JavaScript]
---

# 综述
随着`Nodejs`的出现，`JavaScript`就变得无处不在。前端、后端、混合App、嵌入式设备等。在`Github`上，`JavaScript`标签的项目也是逐年增长，伴随着`ES2015`标准的发布，`JavaScript`也变得`现代`起来。作为一名开发者，不仅要知道`JavaScript`的基础语法，同时也由必要了解`JavaScript`是如何运行的，这里面都牵扯到哪些东西。知道其背后的运行原理，有利于深刻了解我们平常写的代码，同时更好地利用现有的`API`。


# JavaScript引擎
目前最受欢迎的就是Google的`V8`引擎了。`V8`被用在了`Google Chrome`和`Node.js`中。下面的图形简单展示了`JavaScript`引擎的基本工作：
![Engine](js-engine.PNG)

引擎主要负责两部分：
- 内存管理：也就是内存的分配与释放
- 调用栈管理：也就是执行代码时的栈帧管理

# 运行时
在日常开发中，我们还会一些其他的`API`，比如`setTimeout`，这些`API`都是执行在浏览器环境中，这些`API`自然不是`JavaScript引擎`提供的。这些都称之为`Web API`，是由浏览器提供的，除了`setTimeout`，还有操作`DOM`的，`AJAX`等。除去这些还有用来相应`DOM`事件的`事件循环`和`回调队列`。
![browser](browser.PNG)


# 调用栈
`JavaScript`的执行是单线程的，这意味这它只有一个调用栈。`调用栈`说白了就是数据结构，用来记录当前代码执行的一些信息。比如说下面的代码：
```js
function multiply(x, y) {
    return x * y;
}
function printSquare(x) {
    var s = multiply(x, x);
    console.log(s);
}
printSquare(5);
```

当上述的代码开始执行时，`调用栈`是空的。之后开始调用`printSquare`，接着`multiply`，接着`console`，然后开始出栈。形象化表示是这样的：
![CallStack](stack.PNG)

调用栈中的每个条目称之为`栈帧（Stack Frame）`

这里包含了两个情况：
- 异常
- 递归调用

## 异常
```js
function foo() {
    throw new Error('SessionStack will help you resolve crashes :)');
}
function bar() {
    foo();
}
function start() {
    bar();
}
start();
```

如果上述代码运行在`Chrome`中的话，我们可以在`Chrome Del Tools`的`Console`面板中发现会输出一行错误：
![exception](exception.png)

上面的错误直接反映了调用栈的结构。

## 递归调用
递归调用直接导致的后果就是栈溢出，也就是超过了最大栈大小。
```js
function foo() {
    foo();
}
foo();
```
上述代码如果在`Chrome`中，会发现进程会卡死一段时间，然后在`Chrome Del Tools`的`Console`面板中出现错误:
![stack](stackError.png)

调用栈形象化表示是这样的：
![stack](stackOver.png)


# 并发与事件循环
因为`JavaScript`是单线程，所以如果要执行一个耗时的操作，比如复杂的图片转换，就会导致页面出现卡顿现象。此时就无法相应界面的任何操作，导致了非常不好的用户体验。甚至有时会出现以下的提示：
![stuck](stuck.jpeg)

解决这个问题的办法就是使用`异步回调（asynchronous callbacks）`。

这个将在下一篇详细解释。