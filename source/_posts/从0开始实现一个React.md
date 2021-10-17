---
title: 从0开始实现一个React
date: 2021-10-17 12:18:53
tags: react babel
---

react用了这么久还是感觉对生命周期之类的概念理解得很模糊，最近看到一篇[教你构建一个React](https://pomb.us/build-your-own-react/) 的文章，感觉写的很好，而且交互也做得特别棒，在此翻译记录一下。

这篇文章的主要目的是遵循React代码的体系结构一步步地写出一个简单的React，但没有所有的优化和非必要的功能。

过程分为以下几个步骤：

1. The `createElement` Function
2. The `render` Function
3. 并发模型
4. Fibers
5. 渲染和提交阶段
6. 协调
7. 函数组件
8. Hooks

## 准备

首先，`JSX`语法是通过Babel转换成JavaScript的，因此我们需要在项目中配置Babel。

安装：

```shell
npm install --save-dev @babel/core @babel/cli @babel/preset-env @babel/preset-react
```

在根目录中添加`.babelrc`：

```javascript
{
    "presets": ["@babel/preset-env", "@babel/preset-react"]
}
```

`.babel.config.js`

```javascript
const presets = [
  [
    "@babel/env",
    {
      targets: {
        edge: "17",
        firefox: "60",
        chrome: "67",
        safari: "11.1",
      },
      useBuiltIns: "usage",
      corejs: "3.6.4",
    },
  ],
];

module.exports = { presets };
```

运行`npx src --out-dir lib`就可以把转换结果保存在lib文件夹下面了~

## 0：回顾

下面的代码展示了渲染一个React组件的过程：

```javascript
const element = <h1 title="foo">Hello</h1> // 定义React组件
const container = document.getElementById("root") // 从DOM中获取一个节点container
ReactDOM.render(element, container) // 把React组件渲染到container
```

要实现一个React，首先要知道React的方法是如何实现的，用原生的JavaScript怎么写，再去封装成我们的方法。现在，让我们不使用React语法来实现一下这个过程。

1. #### 定义组件

Babel转换`JSX`的规则就是用调用`createElement`替换`<>`内部的代码，将`tag name`、`props`和子元素作为参数传递：

```javascript
const element = React.createElement(
  "h1",
  { title: "foo" },
  "Hello"
)
```

`React.createElement`会对参数做一些验证并返回一个对象，长这样：

```javascript
const element = {
  type: "h1",
  props: {
    title: "foo",
    children: "Hello",
  },
}
```

所以我们可以直接用这个对象替换函数调用。`type`属性是一个字符串，表明DOM节点的类型，就是在生成HTML元素时将要传给`document.createElement`的`tagName`。

`props`属性是一个对象，包括`JSX`属性的所有键值对，还有一个特殊的属性：`children`：

在这个例子中`children`是一个字符串，但是它通常是一个包含多个元素的数组。所以元素实数型结构的。

2. #### 渲染

（`container`是直接调用`document.getElementById`获取的不用管。）另一个我们要替换的React代码是`ReactDOM.render`的调用。`render`是React修改DOM的地方，因此我们要在这里实现DOM更新。

首先我们用`type`属性创建一个节点*，本例中是`h1`。然后我们把元素的元素的属性分配给节点，本例中是title属性。

```javascript
const node = document.createElement(element.type)
node["title"] = element.props.title
```

*为了避免混淆，下文中使用”元素”表示React元素，“节点”表示DOM节点。

接下来我们创建子节点，本例中只有一个文本节点。使用`textNode`而不是`innerText`可以让我们用同样的方式处理所有元素（方便递归）。注意我们像处理`h1`的title一样给文本节点设置`nodeValue`：`{nodeValue: "hello"}`

```javascript
const text = document.createTextNode("")
text["nodeValue"] = element.props.children
```

最后，我们把`textNode`添加到`h1`，再把`h1`添加到`container`。

这样我们就在不使用React的情况下实现了同样的功能。罗嗦了这么多，实现起来还是很简单的，用过React的同学应该都很熟悉：

```javascript
const element = {
  type: "h1",
  props: {
    title: "foo",
    children: "Hello",
  },
}

const container = document.getElementById("root")

const node = document.createElement(element.type)
node["title"] = element.props.title

const text = document.createTextNode("")
text["nodeValue"] = element.props.children

node.appendChild(text)
container.appendChild(node)
```

## 1：`createElement`函数

（上一步中，我们直接定义了element对象）接下来，我们就该真正实现一个自己的`createElement`函数了。首先把`JSX`转换成`JS`研究一下`createElement`是如何调用的：

```javascript
const element = React.createElement(
  "div",
  { id: "foo" },
  React.createElement("a", null, "bar"),
  React.createElement("b")
)
```

还记得我们上一步中定义的element对象吗？我们的函数的功能应该是创建一个具有`type`和`props`属性的对象：

```javascript
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children,
    },
  }
}
```

这里使用了展开运算符处理`props`，使用剩余参数处理`children`，因为我们希望`children`属性是一个数组。例如，`createElement("div")`返回

```javascript
{
  "type": "div",
  "props": { "children": [] }
}
```

`createElement("div", null, a)`返回

```javascript
{
  "type": "div",
  "props": { "children": [a] }
}
```

`createElement("div", null, a, b)`返回

```javascript
{
  "type": "div",
  "props": { "children": [a, b] }
}
```

`children`数组或许包含字符串或数字这样的原始值（树中的叶子节点）。所以我们应该包装不是对象的元素，再给它们一个特殊的属性：`TEXT_ELEMENT`。

当没有子元素时，React并没有包装原始值或创建空数组，但我们更希望简化代码而不是追求代码性能😀。

为了替换React，我们要给自己的库起个名字，就叫`Didact`吧！但是我们依然像使用`JSX`语法，怎么告诉`babel`要用`Didact`的`createElement`而不是`React`的呢？

用这样一个注释，当babel编译`JSX`时，就会使用我们定义的函数辣：

```javascript
/** @jsx Didact.createElement */
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
```

props: {

​      ...props,

​      children: children**.**map(child => 

​        typeof child === 'object'

​          ? child

​          : createTextNode(child)

​      ),

​    },

手残多写了个大括号，一直报错！



requestIdleCallback

**`window.requestIdleCallback()`**方法插入一个函数，这个函数将在浏览器空闲时期被调用。这使开发者能够在主事件循环上执行后台和低优先级工作，而不会影响延迟关键事件，如动画和输入响应。函数一般会按先进先调用的顺序执行，然而，如果回调函数指定了执行超时时间`timeout`，则有可能为了在超时前执行函数而打乱执行顺序。



遍历一颗fiber tree：

一个节点有多个孩子也只有一个child指针

child -> sibling -> uncle





