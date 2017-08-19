---
title: React与Qt WebEngine
date: 2017-08-19 13:29:20
categories: [笔记]
tags: [JavaScript, React, Qt]
---

# 背景介绍
&ensp;&ensp;&ensp;因为一些竞争对手以及技术圈大趋势的原因，我们想要尝试着把未来的产品往前端和移动端发展。但是同时因为长期以来，顾客和大部分工程人员都已经熟悉桌面端的应用。所以考虑到这部分人的顾虑，我们没有采取极端的全B/S的形式，但是同时又不能放弃这种可能性。最后就折衷，采取了`桌面端使用Qt，同时Qt中使用HTML，未来如果全部迁移到Web或者移动端，那么之前的HTML也可以重用。`这样的非主流技术栈。

# React组件如何适配Qt WebEngine
&ensp;&ensp;&ensp;之前做的React组件一直是在Chrome上测试，并且当时以为最后等同事那边的Qt做好以及后台的数据接口改造完后，就直接可以将React组件集成到Qt程序中。

&ensp;&ensp;&ensp;集成的过程中需要关注以下几点：
- JavaScript的模块风格
- React的组件如何导出类给Qt WebEngine使用
- React组件中的CSS


## 模块风格
&ensp;&ensp;&ensp;这个问题很好解决，因为React组件都是使用Webpack打包，只要在output加一句：
```js
libraryTarget: "amd"
```
打包后的bundle文件可以直接在Qt 的对接的文件中使用。

## 导出类
&ensp;&ensp;&ensp;通常如果React组件是直接给HTML用，同时HTML也是用模板。那么唯一需要注意的是，React组件放在哪？所以通常情况下会在HTML中加入`div`节点，同时加上ID，之后在js文件中将React组件加载到这个指定ID的位置。假设有一个名叫`CustomeCalendar`的组件：
```js
ReactDOM.render(
  <CustomeCalendar>,
  document.getElementById('root')
);
```

如果现在是将导出的bundle文件给Qt用，显然不能直接这样进行`render`。唯一的办法是导出一个类给Qt的接口js使用。第一次想到的是这么写：
```js
class CustomCalendar extends Component {
  constructor(props, context) {
    super(props. context)
  }
  ...

  render(){
    return ReactDOM.render(
      ...,
      document.getElementById(this.props.id)
    );
  }
}

export default CustomeCalendar;
```
假设上述React组件的bundle文件就叫`bundle.js`。在Qt中的接口js中就这么使用：
```js
require(['bundle'], function (customCalendar) {
  const cc = new customCalendar('root');
  cc.render();
});
```
在Qt端，运行起来后，控件也显示了。但是稍后就会发现。React库实际上并没有mount这个组件。因为如果`CustomCalendar`组件接收来自外界的数据后，使用`this.setState()`，就会出错。出错的提示类似是这样的：
```js
 setState(...): Can only update a mounted or mounting component. This usually means you called setState() on an unmounted component. This is a no-op. Please check the code for the undefined component.
```

&ensp;&ensp;&ensp;这个问题困扰了我1天时间。直到我大概把React组件的mount的过程弄清楚。以上错误输出来源于`ReactNoopUpdateQueue`，React中还有两类`UpdateQueue`，`ReactUpdateQueue`和`ReactServerUpdateQueue`。正常的组件的`UpdateQueue`想想也应该是`ReactUpdateQueue`。而此处出现的`ReactNoopUpdateQueue`是因为`CustomeCalendar`根本没有被`mount`。如果`CustomeCalendar`没有被`mount`，那么错误信息是谁输出的？答案是`TopLevelWrapper`。React加载任何一个组件前都会创建这个顶层的`TopLevelWrapper`。

再具体的原因在此不表。可以参考这两篇文章：[Under-the-hood-ReactJS](https://bogdan-lyashenko.github.io/Under-the-hood-ReactJS/) 和 [React Internals](http://www.mattgreer.org/articles/react-internals-part-one-basic-rendering/)。

&ensp;&ensp;&ensp;正确的做法是不要直接导出React组件类，而是普通类，之后由Qt的接口js文件创建这个类的实例，最后调用这个实例的渲染方法，在这个方法里进行`ReactDOM.render`。

## CSS
&ensp;&ensp;&ensp;在集成的过程当中出现了在Chrome中显示正确的控件，而在Qt的WebEngine中确显示异常。根本原因尚未弄清，但是直接原因就是css文件在Qt的WebEngine中的解析和Chrome中的不一样。这只能是在Qt端手动去调整了。没有找到更好的办法。

# 总结
&ensp;&ensp;&ensp;JavaScript控件开发还在继续（比如准备用百度的echarts来做一些数据的展示），集成也在继续。相信之后也还会再遇到一些`奇怪`的问题。但是不管怎样，有代码的前提下，掌握调试工具是解决问题的基础。通过调试去了解库或控件，问题才能被清晰定位。反过来因为了解了库或控件的内部机制，这样也能更好地使用它，从而避免一些问题。