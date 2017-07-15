---
title: JavaScript的19个精简语法
date: 2017-06-25 18:29:20
categories: [笔记]
tags: [JavaScript]
---

# 三元操作符简短写法
____Longhand:____
```js

const x = 20;
let big;
if (x > 10) {
    big = true;
} else {
    big = false;
}
```
__Shorthand:__
```js
const big = x > 10 ? true : false;
```

或者嵌套：
```js
const big = x > 10 ? " greater 10" : x < 5 ? "less 5" : "between 5 and 10";
```

个人认为最后一个可读性较差，不建议使用。

# 短路计算简短写法
__Longhand:__
```js
if (variable1 !== null || variable1 !== undefined || variable1 !== '') {
     let variable2 = variable1;
}
```

__Shorthand:__
```js
const variable2 = variable1  || 'new';
```
这个很厉害了，大部分人都不太清楚，至少我自己也是。但是它确实可以写成这样。
```js
let variable1;
let variable2 = variable1  || '';
console.log(variable2 === ''); // prints true
variable1 = 'foo';
variable2 = variable1  || '';
console.log(variable2); // prints foo
```


# 变量声明简短写法
__Longhand:__
```js
let x;
let y;
let z = 3;
```

__Shorthand:__
```js
let x, y, z=3;
```


# If控制语句简短写法
__Longhand:__
```js
if (likeJavaScript === true)
```

__Shorthand:__
```js
if (likeJavaScript)
```

__Longhand:__
```js
let a;
if ( a !== true ) {
// do something...
}
```

__Shorthand:__
```js
let a;
if ( !a ) {
// do something...
}
```


# for控制语句简短写法
__Longhand:__
```js
for (let i = 0; i < allImgs.length; i++)
```

__Shorthand:__
```js
for (let index in allImgs)
```
对于Array：
```js

function logArrayElements(element, index, array) {
  console.log("a[" + index + "] = " + element);
}
[2, 5, 9].forEach(logArrayElements);
// logs:
// a[0] = 2
// a[1] = 5
// a[2] = 9
```


# 短路计算
__Longhand:__
```js
let dbHost;
if (process.env.DB_HOST) {
  dbHost = process.env.DB_HOST;
} else {
  dbHost = 'localhost';
}
```

__Shorthand:__
```js
const dbHost = process.env.DB_HOST || 'localhost';
```


# 基于十进制的指数
__Longhand:__
```js
for (let i = 0; i < 10000; i++) {}
```

__Shorthand:__
```js
for (let i = 0; i < 1e7; i++) {}
// All the below will evaluate to true
1e0 === 1;
1e1 === 10;
1e2 === 100;
1e3 === 1000;
1e4 === 10000;
1e5 === 100000;
```


# 对象属性
以下是对于ES6之后的。

__Longhand:__
```js
const obj = { x:x, y:y };
```

__Shorthand:__
```js
const obj = { x, y };
```

# 箭头函数

这应该属于ES6的语法上的使用

__Longhand:__
```js
function sayHello(name) {
  console.log('Hello', name);
}
setTimeout(function() {
  console.log('Loaded')
}, 2000);
list.forEach(function(item) {
  console.log(item);
});
```

__Shorthand:__
```js
sayHello = name => console.log('Hello', name);
setTimeout(() => console.log('Loaded'), 2000);
list.forEach(item => console.log(item));
```


# 隐示返回值
这也是ES6的箭头函数的语法。

__Longhand:__
```js
function calcCircumference(diameter) {
  return Math.PI * diameter
}
```

__Shorthand:__
```js
calcCircumference = diameter => (
  Math.PI * diameter;
)
```


# 默认参数

ES6的支持。

__Longhand:__
```js
function volume(l, w, h) {
  if (w === undefined)
    w = 3;
  if (h === undefined)
    h = 4;
  return l * w * h;
}
```

__Shorthand:__
```js
volume = (l, w = 3, h = 4 ) => (l * w * h);
volume(2) //output: 24
```


# 模板字面量

ES6的支持。

__Longhand:__
```js
const welcome = 'You have logged in as ' + first + ' ' + last + '.'
const db = 'http://' + host + ':' + port + '/' + database;
```

__Shorthand:__
```js
const welcome = `You have logged in as ${first} ${last}`;
const db = `http://${host}:${port}/${database}`;
```

# 解构赋值

ES6的支持。

__Longhand:__
```js
const observable = require('mobx/observable');
const action = require('mobx/action');
const runInAction = require('mobx/runInAction');
const store = this.props.store;
const form = this.props.form;
const loading = this.props.loading;
const errors = this.props.errors;
const entity = this.props.entity;
```

__Shorthand:__
```js
import { observable, action, runInAction } from 'mobx';
const { store, form, loading, errors, entity } = this.props;
```


# 字符串换行

__Longhand:__
```js
const lorem = 'Lorem ipsum dolor sit amet, consectetur\n\t'
    + 'adipisicing elit, sed do eiusmod tempor incididunt\n\t'
    + 'ut labore et dolore magna aliqua. Ut enim ad minim\n\t'
    + 'veniam, quis nostrud exercitation ullamco laboris\n\t'
    + 'nisi ut aliquip ex ea commodo consequat. Duis aute\n\t'
    + 'irure dolor in reprehenderit in voluptate velit esse.\n\t'
```

__Shorthand:__
```js
const lorem = `Lorem ipsum dolor sit amet, consectetur
    adipisicing elit, sed do eiusmod tempor incididunt
    ut labore et dolore magna aliqua. Ut enim ad minim
    veniam, quis nostrud exercitation ullamco laboris
    nisi ut aliquip ex ea commodo consequat. Duis aute
    irure dolor in reprehenderit in voluptate velit esse.`
```


# …操作符

ES6的支持。

__Longhand:__
```js
// joining arrays
const odd = [1, 3, 5];
const nums = [2 ,4 , 6].concat(odd);
// cloning arrays
const arr = [1, 2, 3, 4];
const arr2 = arr.slice()
```

__Shorthand:__
```js
// joining arrays
const odd = [1, 3, 5 ];
const nums = [2 ,4 , 6, ...odd];
console.log(nums); // [ 2, 4, 6, 1, 3, 5 ]
// cloning arrays
const arr = [1, 2, 3, 4];
const arr2 = [...arr];
```
或者这样：
```js
const odd = [1, 3, 5 ];
const nums = [2, ...odd, 4 , 6];
```
这样：
```js
const { a, b, ...z } = { a: 1, b: 2, c: 3, d: 4 };
console.log(a) // 1
console.log(b) // 2
console.log(z) // { c: 3, d: 4 }
```

在ES6之前可能就要用到Array.concat和Object.Assign，但是没那么方便。

# 强制参数

默认情况下，如果没有对函数的参数传入值，JavaScript将函数的参数设置为undefined，所以很多情况下会对函数的参数
进行检测。

__Longhand:__
```js
function foo(bar) {
  if(bar === undefined) {
    throw new Error('Missing parameter!');
  }
  return bar;
}
```

__Shorthand:__
```js
mandatory = () => {
  throw new Error('Missing parameter!');
}
foo = (bar = mandatory()) => {
  return bar;
}
Array.find
```

__Longhand:__
```js
const pets = [
  { type: 'Dog', name: 'Max'},
  { type: 'Cat', name: 'Karl'},
  { type: 'Dog', name: 'Tommy'},
]
function findDog(name) {
  for(let i = 0; i<pets.length; ++i) {
    if(pets[i].type === 'Dog' && pets[i].name === name) {
      return pets[i];
    }
  }
}
```

__Shorthand:__
```js
pet = pets.find(pet => pet.type ==='Dog' && pet.name === 'Tommy');
console.log(pet); // { type: 'Dog', name: 'Tommy' }
Obejct[key]
```

__Longhand:__
```js
function validate(values) {
  if(!values.first)
    return false;
  if(!values.last)
    return false;
  return true;
}
console.log(validate({first:'Bruce',last:'Wayne'})); // true
```

__Shorthand:__
```js
// object validation rules
const schema = {
  first: {
    required:true
  },
  last: {
    required:true
  }
}
// universal validation function
const validate = (schema, values) => {
  for(field in schema) {
    if(schema[field].required) {
      if(!values[field]) {
        return false;
      }
    }
  }
  return true;
}
console.log(validate(schema, {first:'Bruce'})); // false
console.log(validate(schema, {first:'Bruce',last:'Wayne'})); // true
```

# Double Bitwise NOT

__Longhand:__
```js
Math.floor(4.9) === 4  //true
```
__Shorthand:__
```js
~~4.9 === 4  //true
```