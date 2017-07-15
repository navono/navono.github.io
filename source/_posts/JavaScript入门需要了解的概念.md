---
title: JavaScript入门需要了解的概念
date: 2017-07-08 18:29:20
categories: [笔记]
tags: [JavaScript]
---

# 引言
&ensp;&ensp;&ensp;作为一个一直从事C\C++的程序员来说，进入JavaScript的世界就好像从室内泳池一下子到太平洋的感觉。在C++的世界，一切都比较封闭，你只需要一个文本编辑器，一个编译器或者一个IDE就足够了。然后你就能用你最爱的泳姿“游泳”了。而在JavaScript的世界里，除了前者的文本编辑器，要考虑和了解事情实在是太多了。诸如ES5、ES6、Babel、webpack等，每年都会出现各种框架，各种库，层出不穷，完全可以把人淹没。

写这篇文章的目的主要是为了记录在自己进入这个圈子里，在这个过程中学习到的一些知识，了解了这些基础知识，可能就会少走一些弯路。

# JavaScript的版本
&ensp;&ensp;&ensp;不管何种语言，在进入21世纪后，标准化的工作都进行得越来越频繁。举个例子，拿C++来说，C++98，然后是C++03，之后就是C++11、C++14、C++17、C++20。而且标准委员会对于标准化的进程也做出了比较明确的规划。但是标准的制定和现实中各大编译器厂商的实现之间会有一定的鸿沟。

&ensp;&ensp;&ensp;JavaScript也一样，TC39现在采取的是小步快走的模式，也就是在每一次的标准更新中都只加入少量的特性，但是保持一定的更新频率，而不是一次加入大量的特性，然后好几年更新一次标准。

&ensp;&ensp;&ensp;JavaScript是业界对这门语言的称呼，而在标准制定过程中，是由ECMA这个机构来负责,确切说是ECMA的第39号技术专家委员会（Technical Committee 39，简称TC39）负责制订ECMAScript标准，成员包括Microsoft、Mozilla、Google等大公司。
所以目前的主流是ES6，或者称为ES2015。

## 历史

>1998年6月，ECMAScript 2.0版发布。

>1999年12月，ECMAScript 3.0版发布，成为JavaScript的通行标准，得到了广泛支持。

>2007年10月，ECMAScript 4.0版草案发布，对3.0版做了大幅升级，预计次年8月发布正式版本。草案发布后，由于4.0版的目标过于激进，各方对于是否通过这个标准，发生了严重分歧。以Yahoo、Microsoft、Google为首的大公司，反对JavaScript的大幅升级，主张小幅改动；以JavaScript创造者Brendan Eich为首的Mozilla公司，则坚持当前的草案。

>2008年7月，由于对于下一个版本应该包括哪些功能，各方分歧太大，争论过于激进，ECMA开会决定，中止ECMAScript 4.0的开发，将其中涉及现有功能改善的一小部分，发布为ECMAScript 3.1，而将其他激进的设想扩大范围，放入以后的版本，由于会议的气氛，该版本的项目代号起名为Harmony（和谐）。会后不久，ECMAScript 3.1就改名为ECMAScript 5。
>

>2009年12月，ECMAScript 5.0版正式发布。Harmony项目则一分为二，一些较为可行的设想定名为JavaScript.next继续开发，后来演变成ECMAScript 6；一些不是很成熟的设想，则被视为JavaScript.next.next，在更远的将来再考虑推出。

>2011年6月，ECMAscript 5.1版发布，并且成为ISO国际标准（ISO/IEC 16262:2011）。

>2013年3月，ECMAScript 6草案冻结，不再添加新功能。新的功能设想将被放到ECMAScript 7。

>2013年12月，ECMAScript 6草案发布。然后是12个月的讨论期，听取各方反馈。

>2015年6月17日，ECMAScript 6发布正式版本，即ECMAScript 2015。

&ensp;&ensp;&ensp;以上信息来自于来自[百度百科](http://baike.baidu.com/item/ECMAScript)

&ensp;&ensp;&ensp;上面也提及到，标准的制定和标准的实现是两大阵营。JavaScript的标准实现主要是各大浏览器厂商。而浏览器厂商的实现步伐肯定是落后于标准的制定。但是同时开发者又倾向于在源码中使用新的标准特性，但是用新特性编写的代码在脚本运行环境中又不支持，此时就要谈及到Babel了，一个源码到源码（source-to-source）的转译（transpiller）工具了。


# 运行时环境
&ensp;&ensp;&ensp;目前JavaScript的语言定位是通用型语言，也就是说它不仅仅只是运行在浏览器端，它可以运行在服务端（通过Node.js），运行在移动端（通过React Native），同时也可以创建桌面型应用（通过Electron）。在每个不同的平台上，侧重点是不一样的，所以就导致了应用的一些技术也不同，很明显的就是在服务端和浏览器端的模块化管理。


# 模块化
&ensp;&ensp;&ensp;模块化是每个语言要应用在中大型项目中避免不了的问题。而JavaScript的标准中却迟迟没有对这方面进行标准化。这也就导致了民间出现了很多用于模块化管理的方法或者工具。最常见的也是最主流的有AMD（Asynchronous Module Definition）、CommonJS、UMD（Universal Module Definition）。

&ensp;&ensp;&ensp;三者间的主要区别是：
- AMD，如其名，是异步加载的，主要用在浏览器端
- CommonJs，则是同步加载的，主要用于服务端，Node环境下
- UMD的目标则是想统一上述两者的语法差异

&ensp;&ensp;&ensp;通过下面例子来看看AMD和CommonJS的语法上的区别。
最初的源码，使用的是ES6规范：
```js
export default class Person {
  constructor (name, age) {
    this.name = name;
    this.age = age;
  }

  printInfo () {
    console.log(`Name: {this.age}, age: {this.age}`);
  }
}
```
&ensp;&ensp;&ensp;babel默认情况下产生的输出如下：
```js
"use strict";

Object.defineProperty(exports, "__esModule", {
  value: true
});

var _createClass = function () { function defineProperties(target, props) { for (var i = 0; i < props.length; i++) { var descriptor = props[i]; descriptor.enumerable = descriptor.enumerable || false; descriptor.configurable = true; if ("value" in descriptor) descriptor.writable = true; Object.defineProperty(target, descriptor.key, descriptor); } } return function (Constructor, protoProps, staticProps) { if (protoProps) defineProperties(Constructor.prototype, protoProps); if (staticProps) defineProperties(Constructor, staticProps); return Constructor; }; }();

function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

var Person = function () {
  function Person(name, age) {
    _classCallCheck(this, Person);

    this.name = name;
    this.age = age;
  }

  _createClass(Person, [{
    key: "printInfo",
    value: function printInfo() {
      console.log("Name: {this.age}, age: {this.age}");
    }
  }]);

  return Person;
}();

exports.default = Person;
```
&ensp;&ensp;&ensp;这其实是CommonJS的语法格式。如果想要生成AMD格式的，需要借助babel-plugin-transform-es2015-modules-amd插件。输出如下：
```js
define(["exports"], function (exports) {
  "use strict";

  Object.defineProperty(exports, "__esModule", {
    value: true
  });

  function _classCallCheck(instance, Constructor) {
    if (!(instance instanceof Constructor)) {
      throw new TypeError("Cannot call a class as a function");
    }
  }

  var _createClass = function () {
    function defineProperties(target, props) {
      for (var i = 0; i < props.length; i++) {
        var descriptor = props[i];
        descriptor.enumerable = descriptor.enumerable || false;
        descriptor.configurable = true;
        if ("value" in descriptor) descriptor.writable = true;
        Object.defineProperty(target, descriptor.key, descriptor);
      }
    }

    return function (Constructor, protoProps, staticProps) {
      if (protoProps) defineProperties(Constructor.prototype, protoProps);
      if (staticProps) defineProperties(Constructor, staticProps);
      return Constructor;
    };
  }();

  var Person = function () {
    function Person(name, age) {
      _classCallCheck(this, Person);

      this.name = name;
      this.age = age;
    }

    _createClass(Person, [{
      key: "printInfo",
      value: function printInfo() {
        console.log("Name: {this.age}, age: {this.age}");
      }
    }]);

    return Person;
  }();

  exports.default = Person;
});
```
&ensp;&ensp;&ensp;随着ES6标准的发布，在ES6中已经原生支持了模块化的功能。所有可以在代码中使用ES6的原生模块化的语法。

&ensp;&ensp;&ensp;有时我们需要关注产出物的模块管理方式，这样在使用的时候才不会出现各种问题。Babel只做了源码转译的工作，每一个输入文件对应一个输出文件，所以我们需要一个将这些输出文件打包成一个文件的工具，也就是webpack。


# Babel
&ensp;&ensp;&ensp;[Babel](https://babeljs.io/)是一个让开发者可以用新特性编写代码，同时又能运行的一个工具。主要是弥补上述提及的新标准和浏览器支持度之间的鸿沟。它的主要设计思想是插件化。

&ensp;&ensp;&ensp;Babel有几种运行方式：
- 命令行的方式，通过babel-cli
- 运行在node环境下，通过babel-node
- 运行在源码中，通过babel-register

&ensp;&ensp;&ensp;在上面章节中，我们使用了插件babel-plugin-transform-es2015-modules-amd来生成AMD格式的代码。使用插件的方式可以通过命令行，比如：
```js
babel --plugins transform-es2015-modules-amd .\src.js -o dst.js
```

&ensp;&ensp;&ensp;或者也可以使用.babelrc配置文件来进行配置，比如：
```js
{
  "presets": ["es2015"],
  "plugins": ["transform-es2015-modules-amd"]
}
```
&ensp;&ensp;&ensp;还有一些配置可以参照官方文档。


# Webpack
&ensp;&ensp;&ensp;[Webpack](https://webpack.js.org/)是个打包工具，同时也会管理模块的依赖关系。是目前比较主流的打包工具。webpack本身也是使用模块化的方法去打包，同时也是插件化的组织方式。具有以下几个特点：
- 代码拆分（code splitting）
- 静态分析
- 模块热替换（Hot Module Replacement）

&ensp;&ensp;&ensp;打包后的文件可直接被使用。但是同时有一点需要注意的是，就是打包后的文件格式采取的模块化管理方式。为了将bundle适应不同的运行时环境，通过output.libraryTarget选项，webpack支持将bundle打包成不同格式的模块化代码。具体设置可以参考[此链接](https://webpack.js.org/configuration/output/#output-librarytarget)。