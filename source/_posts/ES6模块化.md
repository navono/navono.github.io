---
title: ES6模块化
date: 2017-08-12 18:29:20
categories: [笔记]
tags: [JavaScript]
---

# 概括
JavaScript一直没有在语言层面支持代码的模块化，而是一直通过库来实现。ES6是第一次在标准在引入内建的模块化。

# JavaScript中的模块化
ES6的模块块集成了之前已有的模块化的大部分有点，包括：
- 每个模块都是加载后执行的代码块
- 在代码块中，其内的声明（变量声明，函数声明）满足：
  - 默认情况下，这些变量的作用域都只在模块内部。
  - 可以在一些变量前加入export，将变量导出。
- 一个模块可以从其他模块导入声明。有以下几种方式：
  - 通过相对路径（'../model/helpers'），文件后缀.js可以省略。
  - 通过绝对路径（'/lib/js/healpers'），直接指向要导入的模块
  - 通过命名方式（'util'），模块名指向的是一个已经配置的名字。
- 模块都是单例的。即使一个模块导入了多次，只会有一个模块的“实例”存在。

## ES5的模块化
ES5中存在两种模块化的方式，这两种方式是不兼容的。
- CommonJS 模块化。这个标准的主要实现者是Node.js（但是Node.js的模块化的一些特性要超越CommonJS），有以下特性：
  - 紧凑的语法
  - 用于同步加载和服务器端
- Asynchronous Module Definition (AMD)。标准的实现主要是RequireJS，有以下特性：
  - 稍微复杂的语法，不需要eval()就能够使AMD工作
  - 用于异步加载和浏览器端

## ES6的模块化
ES6模块化设计需要融合了上述两者的优点：
- 和CommonJS一样有紧凑的语法，偏向于单导出和支持循环依赖
- 和AMD一样，可直接支持异步模式加载和可配置模式的模块加载

但是同时ES6却要超越上述两者：
- 支持语法比CommonJS更紧凑
- 支持静态分析（静态检测、优化等）
- 支持比CommonJS更好的循环依赖

ES6模块化标准包括两个部分：
- 声明式语法（导入和导出）
- 编程式的加载器API，用于配置模块如何加载和模块有条件地加载。

# ES6模块化的基本知识
ES6支持两种方式的导出，可以同时使用，也可以独立使用，通常情况下是分开使用：
- 多个命名的导出（一个模块多个导出）
- 单个默认的导出（一个模块一个导出）

## 默认导出

### 导出方式
- 导出标签式声明
- 直接导出 *默认导出(Default-exporting)* 值

### import和export必须在文件顶部
因为ES6模块的结构是静态的，所以不能有条件地导入或者导出。
```js
if (Math.random()) {
    import 'foo'; // SyntaxError
}

// You can’t even nest `import` and `export`
// inside a simple block:
{
    import 'foo'; // SyntaxError
}
```

### import会被提升
下面的代码会被正确运行：
```js
foo();

import { foo } from 'my_module';
```

### import的东西在使用方被视为只读
```js
//------ lib.js ------
export let counter = 3;
export function incCounter() {
    counter++;
}

//------ main.js ------
import { counter, incCounter } from './lib';

// The imported value `counter` is live
console.log(counter); // 3
incCounter();
console.log(counter); // 4
```

导入的东西被视为视图有以下好处：
- 支持循环依赖，即使是非限定（unqualified ）的导入
- 限定（Qualified ）导入和非限定（支持循环依赖，即使是非限定（unqualified）导入以同样的方式运行
- 将代码切分到多个模块也可正常运行（只要不修改导出的值）

### 支持循环依赖

#### CommonJS中的循环依赖
TBD

#### ES6循环依赖
TBD

# 导入与导出
## 导出风格
- 默认导出
  ```js
  import localName from 'src/my_lib';
  ```
- 命名空间导出：已对象的方式导出模块
  ```js
  import * as my_lib from 'src/my_lib';
  ```
- 具名导出
  ```js
  import { name1, name2 } from 'src/my_lib';
  ```
  或者：
  ```js
   // Renaming: import `name1` as `localName1`
  import { name1 as localName1, name2 } from 'src/my_lib';

  // Renaming: import the default export as `foo`
  import { default as foo } from 'src/my_lib';
  ```
- 空导入。只加载模块，不导入任何东西。会执行模块的代码体
  ```js
  import 'src/my_lib';
  ```
有两种方式来组合上述的导出：
- 默认导出和命名空间导出的组合
  ```js
  import theDefault, * as my_lib from 'src/my_lib';
  ```
- 默认导出和具名导出的组合
  ```js
  import theDefault, { name1, name2 } from 'src/my_lib';
  ``` 

## 具名导出风格：inline vs. clause
一种风格是在声明前直接加export关键字
```js
export var myVar1 = ···;
export let myVar2 = ···;
export const MY_CONST = ···;

export function myFunc() {
    ···
}
export function* myGeneratorFunc() {
    ···
}
export class MyClass {
    ···
}
```
另外一种是在模块的末尾同一导出：
```js
const MY_CONST = ···;
function myFunc() {
    ···
}

export { MY_CONST, myFunc };
```
也可以以不同的名字导出：
```js
export { MY_CONST as FOO, myFunc };
```

## 所有导出风格
ES6支持一下几种导出风格：
- 再导出
  - 再导出所有
  ```js
  export * from 'src/other_module';
  ```
  - 通过子句（clause）再导出
  ```js
  export { foo as myFoo, bar } from 'src/other_module';
  export { default } from 'src/other_module';
  export { default as foo } from 'src/other_module';
  export { foo as default } from 'src/other_module'
  ```
- 通过子句（clause）具名导出
  ```js
  export { MY_CONST as FOO, myFunc };
  export { foo as default };
  ```
- 内联（inline）具名导出
  - 变量声明方式
    ```js
    export var foo;
    export let foo;
    export const foo;
    ```
  - 函数声明方式
    ```js
    export function myFunc() {}
    export function* myGenFunc() {}
    ```
  - 类声明方式
    ```js
    export class MyClass {}
    ```
- 默认导出
  - 函数声明方式
    ```js
    export default function myFunc() {}
    export default function () {}

    export default function* myGenFunc() {}
    export default function* () {}
    ```
  - 类声明方式
    ```js
    export default class MyClass {}
    export default class {}
    ```
  - 表达式（注意分号）
    ```js
    export default foo;
    export default 'Hello world!';
    export default 3 * 7;
    export default (function () {});
    ```

### 建议不要混用默认导出和具名导出
### 默认导出只是另外一种具名导出
下面代码是等同的：
```js
import { default as foo } from 'lib';
import foo from 'lib';
```
```js
//------ module1.js ------
export default function foo() {} // function declaration!

//------ module2.js ------
function foo() {}
export { foo as default };
```

### default可以作为导出的名字，但是不能作为变量名
这就意味着default只能出现在别名导入的左边
```js
import { default as foo } from 'some_module';
```
或则别名导出的右边：
```js
export { foo as default };
```
或则再导出：
```js
export { myFunc as default } from 'foo';
export { default as otherFunc } from 'foo';

// The following two statements are equivalent:
export { default } from 'foo';
export { default as default } from 'foo';
```

## 在使用方，导入的是视图
CommonJS和ES6导入方面有明显的不同：
- 在CommonJS中，导入的值只是导出的一份拷贝
- 在ES6中，导入的值是导出值的一个只读视图

CommonJS环境中：
```js
//------ lib.js ------
var counter = 3;
function incCounter() {
    counter++;
}
module.exports = {
    counter: counter, // (A)
    incCounter: incCounter,
};

//------ main1.js ------
var counter = require('./lib').counter; // (B)
var incCounter = require('./lib').incCounter;

// The imported value is a (disconnected) copy of a copy
console.log(counter); // 3
incCounter();
console.log(counter); // 3

// The imported value can be changed
counter++;
console.log(counter); // 4
```

ES6环境中：
```js
//------ lib.js ------
export let counter = 3;
export function incCounter() {
    counter++;
}

//------ main1.js ------
import { counter, incCounter } from './lib';

// The imported value `counter` is live
console.log(counter); // 3
incCounter();
console.log(counter); // 4

// The imported value can’t be changed
counter++; // TypeError
```
使用*导入也一样：
```js
//------ main2.js ------
import * as lib from './lib';

// The imported value `counter` is live
console.log(lib.counter); // 3
lib.incCounter();
console.log(lib.counter); // 4

// The imported value can’t be changed
lib.counter++; // TypeError
```
但是有一点需要注意的是不能修改导入的对象，但是可以修改对象引用的属性。
```js
//------ lib.js ------
export let obj = {};

//------ main.js ------
import { obj } from './lib';

obj.prop = 123; // OK
obj = {}; // TypeError
```