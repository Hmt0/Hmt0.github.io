---
title: 从0开始实现一个React
date: 2021-10-17 12:18:53
tags: react babel
---

最近看到一篇[教你构建一个React](https://pomb.us/build-your-own-react/) 的文章，感觉写的很好，而且交互也做得特别棒，在此翻译记录一下。

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

## -1：准备

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

运行`npx babel src --out-dir lib`就可以把转换结果保存在lib文件夹下面了~

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

(这里的箭头函数我一开始多写了个大括号，然后就一直报错！)

```javascript
props: {
      ...props,
      children: children.map(child =>
        typeof child === "object"
          ? child
          : createTextElement(child)
      ),
    },
  }
}

function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: [],
    },
  }
}
```

当没有子元素时，React并没有包装原始值或创建空数组，但我们更希望简化代码而不是追求代码性能😀。

为了替换React，我们要给自己的库起个名字，就叫`Didact`吧！但是我们依然想使用`JSX`语法，怎么告诉`babel`要用`Didact`的`createElement`而不是`React`的呢？

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

## 2.`render`函数

接下来，我们要写一个自己的`ReactDOM.render`函数。现在呢，我们只关心向DOM中添加东西，之后我们再实现更新和删除。

我们先根据`element type`生成一个DOM节点，再把这个新节点添加到container。

我们要递归地对每个子节点执行相同的操作。别忘了文本元素，如果元素类型是`TEXT_ELEMENT`我们就会创建一个文本节点而不是常规节点：

```javascript
function render(element, container) {
  const dom =
    element.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(element.type)

  element.props.children.forEach(child =>
    render(child, dom)
  )
```

最后我们需要给节点分配元素属性：

```javascript
const isProperty = key => key !== "children"
  Object.keys(element.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = element.props[name]
    })
```

完成！我们实现了一个可以把`JSX`渲染到DOM中的库。

## 3.并发模型

在实现更多功能前······我们需要重构一下代码。

因为我们的递归调用有一个问题。一旦我们开始渲染，直到我们渲染完整棵element树都不会停下来。如果这棵element树很大的话，它可能会使主线程阻塞很久。而且如果浏览器需要实现一段平滑的动画或者处理用户输入这种高优先级的任务，它将会一直等待直到渲染完成。

因此我们将把工作拆分成小的单元，而且每当我们完成了一个单元，如果有其他事情要做的话我们会让浏览器打断渲染。

```javascript
let nextUnitOfWork = null

function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) { 
    // 任务列表中还有下一个单元而且还有空闲时间
    nextUnitOfWork = performUnitOfWork(
      // 执行任务
      nextUnitOfWork
    )
    shouldYield = deadline.timeRemaining() < 1
   	// 判断是否还有空闲时间
  }
  requestIdleCallback(workLoop)
}

requestIdleCallback(workLoop)

function performUnitOfWork(nextUnitOfWork) {
  // TODO
}
```

我们使用`requestIdleCallback`来实现循环。可以把`reuqestIdleCallback`看作一个`setTimeout	`,只不过不是我们告诉他什么时候运行，**浏览器会在主线程空闲时运行回调函数**。

React不再使用`requestIdleCallback`了，现在人家用*the scheduler package*，但在本例中是一样的概念。

`requestIdleCallback`还给了我们一个deadline参数。我们就是用它来判断在浏览器获得掌控权之前我们还有多少时间的。

我们通过设置任务中的第一个单元来启动循环，接下来写了一个`performUnitOfWork`函数，即执行了任务又返回了任务中的下一个单元。

## 4.`Fibers`

（现在我们还不知道`nextUnitOfWork`长啥样，也就无法实现`performUnitOfWork`）为了组织任务中的所有单元我们需要一种数据结构：那就是`fiber`树。

（Fiber就是我们经常说的”虚拟DOM“，其实就是先用一个对象来表述这个节点的自身属性和它的父亲儿子兄弟的关系）

**每一个元素都有一个对应的`fiber`而且每个`fiber`都是任务中的一个单元。**

让我们来看一个例子，假设我们想要渲染一个这样的element树：

```javascript
Didact.render(
  <div>
    <h1>
      <p />
      <a />
    </h1>
    <h2 />
  </div>,
  container
)
```

在`render`中我们将会创建一个`root fiber`，并把它设置为`nextUnitOfWork`。接下来的任务将会在`performUnitOfWork`中执行，我们将会对每个fiber做三件事情：

1. 把元素添加到DOM中
2. 为元素的孩子创建`fibers`
3. 选择下一个任务

（不知道你有没有看出来，其实还是递归，只不过我们没有在递归里面不假思索地渲染每一个节点！）

学过数据结构的同学都应该知道树长啥样：

![image-20211018223652632](C:\Users\luna6\AppData\Roaming\Typora\typora-user-images\image-20211018223652632.png)

就长fiber树这样！其实是fiber树就长树这样😀选择这种数据结构当然是为了方便遍历了，每个fiber都链接着它的第一个孩子节点，相邻的兄弟节点和它的父节点。

当我们完成了一个fiber上的任务时，如果他有`child`节点，那么孩子fiber就会是任务执行的下一个单元。例如，当我们完成了`div`fiber上的任务时，下一个单元将会是`h1`fiber。

如果一个fiber没有`child`，我们把`sibling`当作任务的下一个单元。比如，`p`fiber没有`child`，所以我们在它完成之后来到了`a`fiber。

再如果一个fiber既没有`child`也没有`sibling`呢，我们找到它的“叔叔”：`parent`的`sibling`。就像`a`fiber和`h2`fiber的关系。

当然啦，如果`parent`也没有`sibling`的话，我们就一直顺着`parent`s向上找，直到找到了一个具有`sibling`的fiber或者到达了root。如果我们到达了root，也就意味着我们完成了此次`render`的所有任务。（熟悉数据结构的同学一定很开心吧！这不就是先根遍历的过程嘛😄）

接下来让我们实现代码部分：

首先把render函数中创建DOM节点部分的代码抽出来，放在一个单独的`createDom`函数中，稍后我们会用到它：

```javascript
function createDom(fiber) {
  const dom =
    fiber.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(fiber.type)
​
  const isProperty = key => key !== "children"
  Object.keys(fiber.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = fiber.props[name]
    })
​
  return dom
}
```

在`render`函数中我们把fiber tree的root设置为`nextUnitOfWork`

```javascript
function render(element, container) {
  nextUnitOfWork = {
    dom: container,
    props: {
      children: [element],
    },
  }
}
```

接下来，当浏览器准备好时，它会回调我们的`workLoop`，我们将会从`root`开始执行任务。首先，我们创建一个新节点并把它添加到DOM中。我们用`fiber.dom`属性来跟踪这个DOM节点：

```javascript
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }

  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }

  // TODO create new fibers
  // TODO return next unit of work
}
```

接下来我们为每个`child`创建一个新的fiber：

```javascript
const elements = fiber.props.children
  let index = 0
  let prevSibling = null

  while (index < elements.length) {
    const element = elements[index]

    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }
  }
```

然后我们把它作为一个`child`或者`sibling`加入fiber tree，这取决于它是不是第一个`child`：

```javascript
if (index === 0) {
      fiber.child = newFiber
    } else {
      prevSibling.sibling = newFiber
    }

    prevSibling = newFiber
    index++
```

最后我们寻找任务中的下一个单元。我们首先寻找`child`，接下来是`sibling`，然后是`uncle`，以此类推。

```javascript
 if (fiber.child) {
    return fiber.child
  }
  let nextFiber = fiber
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }
```

这就是我们的`performUnitOfWork`辣！





