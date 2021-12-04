---
title: 从0开始实现一个React下
date: 2021-10-20 23:00:10
tags: react
---

书接上回，我们已经实现了`createElement`函数、`render`函数和Fibers，接下来就要开始处理组件更新的部分了~

## 5.`Render`和`Commit`阶段

但在这之前！不知道你有没有注意到一个问题：在`performUnitOfWork`中每次我们处理一个元素时都会在DOM 中添加一个新的节点。而且，浏览器还会在完成整棵树的渲染之前打断我们的工作。这样，用户将会看到一个不完整的`UI`，这是我们不希望的。

所以我们把修改DOM的这部分代码移除：

```javascript
function performUnitOfWork(fiber) {
    console.log("performUnitOfWork", fiber)
    // TODO add dom node
    if(!fiber.dom) {
        fiber.dom = createDom(fiber)
    }
    // TODO create new fibers
    const elements = fiber.props.children
    let index = 0
    let prevSibling = null
...
```

并且在render函数中去跟踪fiber树的根节点，我们叫它“正在进行的工作”root或者`wipRoot`。

当我们完成了所有任务（也就是任务中没有下一个单元的时候），我们就把整个fiber树提交到DOM中：

```javascript
function commitRoot() {
  // TODO add nodes to dom
}

function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    },
  }
  nextUnitOfWork = wipRoot
}

let nextUnitOfWork = null
let wipRoot = null

function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    )
    shouldYield = deadline.timeRemaining() < 1
  }

  if (!nextUnitOfWork && wipRoot) {
    commitRoot()
  }

  requestIdleCallback(workLoop)
}
```

我们在`commitRoot`函数中完成提交。这里我们递归地将所有节点添加到DOM中：

```javascript
function commitRoot() {
  commitWork(wipRoot.child)
  wipRoot = null
}

function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom
  domParent.appendChild(fiber.dom)
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

（也就是不断执行`requestIdleCallback(workLoop)`生成fiber树，修改DOM的操作集中在commit阶段）

## 6.协调

带目前为止我们只是在DOM中添加东西，怎么去更新或者删除节点呢？

这就是所谓的协调（reconciliation）了，我们需要将在`render`函数上接收到的元素与提交给DOM的最后一个fiber树进行比较。

所以完成提交后，我们需要保存“上一个提交到DOM的fiber树”的引用。我们称它为`currentRoot`。

我们还要为每一个fiber添加一个`alternate`属性。这个属性是到到旧fiber的链接，也就是我们在上一次提交阶段中提交到DOM中的fiber。

```javascript
function commitRoot() {
	...
	currentRoot = wipRoot
}

funciton render(element, container) {
	wipRoot = {
		...
		alternate: currentRoot
	}
}

let currentRoot = null
```

现在让我们把创建新fibers的代码从`performUnitOfWork`中提取出来，放在一个新的`reconcileChildren`函数中。在这里我们把旧fibers和新元素调和起来：

```javascript
function performUnitOfWork(fiber) {
    ...
    const elements = fiber.props.children
    reconcileChildren(fiber, element)
    ...
}
function reconcileChildren(wipFiber, elements) {}
```

我们同时迭代旧fiber的孩子（`wipFiber.alternate`）和我们想调和的元素数组。

如果忽略同时遍历一个数组和链表的模板代码，在while循环中只剩下最重要的部分：`oldFiber`和`element`。**`element`是我们想渲染在DOM中的东西，`oldFiber`则是我们上次的渲染的内容。**

我们需要比较它们，来看看是不是有需要应用在DOM中的更改：

```javascript
let index = 0
  let oldFiber =
    wipFiber.alternate && wipFiber.alternate.child
  let prevSibling = null

  while (
    index < elements.length ||
    oldFiber != null
  ) {
    const element = elements[index]
    let newFiber = null

    // TODO compare oldFiber to element
  }
```

我们使用类型来比较它们：

- 如果旧fiber和新元素类型一样，我们可以保留DOM节点并且只更新的属性
- 如果类型不同，并且存在新元素，意味着我们需要创建一个新的DOM节点
- 如果类型不同，并且存在旧fiber，我们需要移除旧节点

在这里React还用了keys，来实现更高效的调和。比如，它检测子元素在元素数组中的位置何时改变。

```javascript
  const sameType =
      oldFiber &&
      element &&
      element.type == oldFiber.type

    if (sameType) {
      // TODO update the node
    }
    if (element && !sameType) {
      // TODO add this node
    }
    if (oldFiber && !sameType) {
      // TODO delete the oldFiber's node
    }
```

当旧fiber和元素有相同的类型时，我们创建一个新fiber，使DOM节点和旧fiber保持一致，属性和元素保持一致。

我们还给fiber添加了一个新的属性：`effectTag`。接下来，我们将在提交阶段使用这个属性：

```javascript
const sameType =
      oldFiber &&
      element &&
      element.type == oldFiber.type

    if (sameType) {
      newFiber = {
        type: oldFiber.type,
        props: element.props,
        dom: oldFiber.dom,
        parent: wipFiber,
        alternate: oldFiber,
        effectTag: "UPDATE",
      }
    }
```

对于需要一个新的DOM节点的元素，我们用`PLACEMENT`来标记新fiber：

```javascript
if (element && !sameType) {
      newFiber = {
        type: element.type,
        props: element.props,
        dom: null,
        parent: wipFiber,
        alternate: null,
        effectTag: "PLACEMENT",
      }
    }
```

对于需要删除节点的情况，我们不需要新fiber，只要在旧fiber上添加effect tag就好了：

```javascript
 if (oldFiber && !sameType) {
      oldFiber.effectTag = "DELETION"
      deletions.push(oldFiber)
    }
```

但是，当我们把fiber树提交到DOM时，我们从“正在工作中的”根执行，它并没有旧fibers呀。

因此我们需要一个数组deletions来跟踪想要移除的节点。接着，当我们向DOM提交更改时，我们仍然使用deletions数组中的fibers：

```javascript
function commitRoot() {
  deletions.forEach(commitWork)
  commitWork(wipRoot.child)
  currentRoot = wipRoot
  wipRoot = null
}
```

现在，让我们修改`commitWork`函数来处理新的`effectTags`。

```javascript
function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom
  domParent.appendChild(fiber.dom)
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

如果fiber的effect tag值是`PLACEMENT`，我们和之前做的一样，把DOM节点追加到父fiber的节点上；

如果是`DELETION`,则相反，我们要删除子节点；

如果是`UPDATE`，我们需要用更改后的属性更新现存的DOM节点：

```javascript
function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom
   if (
    fiber.effectTag === "PLACEMENT" &&
    fiber.dom != null
   ) {
       domParent.appendChild(fiber.dom)
   } else if (
    fiber.effectTag === "UPDATE" &&
    fiber.dom != null
   ) {
       updateDom(
           fiber.dom,
           fiber.alternate.props,
           fiber.props
       ) 
   } else if (fiber.effectTag === "DELETION") {
     domParent.removeChild(fiber.dom)
   } 
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

我们把更新的操作写在`updateDom`函数中：我们对比新旧fibers的属性，移除无用属性，设置新的或改变的属性：

```javascript
const isProperty = key => key !== "children"
const isNew = (prev, next) => key =>
  prev[key] !== next[key]
const isGone = (prev, next) => key => !(key in next)
function updateDom(dom, prevProps, nextProps) {
  // Remove old properties
  Object.keys(prevProps)
    .filter(isProperty)
    .filter(isGone(prevProps, nextProps))
    .forEach(name => {
      dom[name] = ""
    })

  // Set new or changed properties
  Object.keys(nextProps)
    .filter(isProperty)
    .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      dom[name] = nextProps[name]
    })
}
```

对于事件监听属性我们需要做特殊处理，也就是属性名以“on”前缀开头的属性：

```javascript
const isEvent = key => key.startsWith("on")
const isProperty = key =>
  key !== "children" && !isEvent(key)
```

如果事件处理程序发生了改变，我们把它从节点中移除：

```javascript
  //Remove old or changed event listeners
  Object.keys(prevProps)
    .filter(isEvent)
    .filter(
      key =>
        !(key in nextProps) ||
        isNew(prevProps, nextProps)(key)
    )
    .forEach(name => {
      const eventType = name
        .toLowerCase()
        .substring(2)
      dom.removeEventListener(
        eventType,
        prevProps[name]
      )
    })
```

然后添加一个新的处理程序：

```javascript
 // Add event listeners
  Object.keys(nextProps)
    .filter(isEvent)
    .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      const eventType = name
        .toLowerCase()
        .substring(2)
      dom.addEventListener(
        eventType,
        nextProps[name]
      )
    })
```

（其实就是初次渲染的时候要从头（根）到尾（叶）生成一棵fiber树，之后有更新只需要调整部分节点，保留不变的节点）

## 7.函数组件

接下来我们要做的是支持函数式组件。首先让我们把原来的例子修改成函数式组件，它返回一个`h1`元素：

```javascript
/** @jsx Didact.createElement */
function App(props) {
  return <h1>Hi {props.name}</h1>
}
const element = <App name="foo" />
const container = document.getElementById("root")
Didact.render(element, container)
```

如果把JSX语法转换成JS呢，应该是下面这样的:

```javascript
function App(props) {
  return Didact.createElement(
    "h1",
    null,
    "Hi ",
    props.name
  )
}
const element = Didact.createElement(App, {
  name: "foo",
})
```

函数式组件有两个不同点：

- 函数式组件的fiber（本身）没有DOM节点（本身是由其他由DOM节点的fiber组成的嘛）
- 而且组件的孩子来自于函数的执行而不是直接来自于`props`

我们检查fiber type是不是一个函数，然后据此切换到一个不同的更新函数。

```javascript
const isFunctionComponent =
    fiber.type instanceof Function
  if (isFunctionComponent) {
    updateFunctionComponent(fiber)
  } else {
    updateHostComponent(fiber)
```

在`updateHostComponent`中我们做的事和之前一样：

```java
function updateFunctionComponent(fiber) {
  // TODO
}

function updateHostComponent(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
  reconcileChildren(fiber, fiber.props.children)
}
```

而在`updateFunctionComponent`中我们运行函数来获得children。

在我们的例子中，这里的`fiber.type`是`App`函数，当我们运行它时会返回一个`h1`元素。

接着，一旦我们得到了children，调和函数也会以同样的方式执行，我们不需要对它做任何修改

```javascript
function updateFunctionComponent(fiber) {
  const children = [fiber.type(fiber.props)]
  reconcileChildren(fiber, children)
}
```

我们需要修改的是`commitWork`函数，现在我们的到了一些没有DOM节点的fibers，我们要修改两件事。

首先，为了找到DOM节点的父亲，我们需要在fiber tree中向上查找，直到找到了拥有DOM节点的fiber：

```javascript
 let domParentFiber = fiber.parent
  while (!domParentFiber.dom) {
    domParentFiber = domParentFiber.parent
  }
  const domParent = domParentFiber.dom

  if (
    fiber.effectTag === "PLACEMENT" &&
    fiber.dom != null
  ) {
    domParent.appendChild(fiber.dom)
  }
```

接着当删除节点时，我们也需要继续查找，直到找到一个具有DOM节点的子元素：

```javascript
function commitDeletion(fiber, domParent) {
  if (fiber.dom) {
    domParent.removeChild(fiber.dom)
  } else {
    commitDeletion(fiber.child, domParent)
  }
}
```

## 8.`Hooks`

最后一步了！现在我们实现了函数式组件，现在让我们加入状态。

让我们写一个经典的计数器例子。每次我们点击它，它的state就加一。注意我们用的是`Diadact.useState`来获取和更新counter值：

```javascript
function Counter() {
  const [state, setState] = Didact.useState(1)
  return (
    <h1 onClick={() => setState(c => c + 1)}>
      Count: {state}
    </h1>
  )
}
const element = <Counter />
```

`updateFunctionComponent`是我们调用示例中`Counter`函数的地方，在**函数内部**我们还调用了`useState`：

```javascript
function updateFunctionComponent(fiber) {
  const children = [fiber.type(fiber.props)]
  reconcileChildren(fiber, children)
}

function useState(initial) {
  // TODO
}
```

在调用函数组件之前，我们需要初始化一些全局变量，供我们在`useState`函数中使用。

首先，我们定义“正在工作的”fiber。

我们还在fiber中添加了`hooks`数组，来实现在一个组件中多次调用`useState`。同时我们跟踪当前的hook下标：

```javascript
let wipFiber = null
let hookIndex = null

function updateFunctionComponent(fiber) {
  wipFiber = fiber
  hookIndex = 0
  wipFiber.hooks = []
  const children = [fiber.type(fiber.props)]
  reconcileChildren(fiber, children)
}
```

当函数式组件调用`useState`时，我们检查是否有一个旧hook。我们用hook下标在fiber的`alternate`属性中检查。

如果我们有一个旧hook，就把旧hook中的state复制到新hook中，没有的话呢就初始化state。

接着我们把新hook添加到fiber，把hook下标加一，并且返回state：

```javascript
function useState(initial) {
  const oldHook =
    wipFiber.alternate &&
    wipFiber.alternate.hooks &&
    wipFiber.alternate.hooks[hookIndex]
  const hook = {
    state: oldHook ? oldHook.state : initial,
  }

  wipFiber.hooks.push(hook)
  hookIndex++
  return [hook.state]
}
```

`useState`还应该返回一个更新state的函数，所以我们定义一个`setState`函数来接收一个动作（在`Counter`例子中这个动作是把state加一的函数）。

我们把这个动作推入一个队列，这个队列是添加在hook上的。

接下来我们要做的事和之前在`render`函数中的类似，设置一个新的“工作中的”root作为任务的下一个单元，使workLoop能开启一个新的渲染阶段：

```javascript
const hook = {
    state: oldHook ? oldHook.state : initial,
    queue: [],
  }

  const setState = action => {
    hook.queue.push(action)
    wipRoot = {
      dom: currentRoot.dom,
      props: currentRoot.props,
      alternate: currentRoot,
    }
    nextUnitOfWork = wipRoot
    deletions = []
  }
​
  wipFiber.hooks.push(hook)
  hookIndex++
  return [hook.state, setState]
}
```

我们还没执行动作呢！

我们在**下次渲染组件的时候**再执行，我们从旧hook队列中拿到所有动作，再把他们一个个应用在新hook state上，所以当我们返回state的时候它已经更新过了：

```javascript
const actions = oldHook ? oldHook.queue : []
actions.forEach(action => {
    hook.state = action(hook.state)
})
```

大功告成！我们写了一个自己的React😀

下面我们来复习一下整个过程8：

render:

![image-20211028180235291](C:\Users\luna6\AppData\Roaming\Typora\typora-user-images\image-20211028180235291.png)

hookindex是干嘛的？

每次触发一个更新，调用setState函数，都会更新nextUnitOfWork，开始新一轮的重新渲染

此时fiber一定是函数组件，index在更新函数中初始化为0

生成children的就是执行函数的过程

会挨个执行useState函数

初始化的时候不执行setState的回调

但是在之后的更新中，需要拿到上一个fiber状态中保存的对应useState,setState回调队列，因为不是每一个setState都执行

可能再一次事件中调用了多个setState

在执行setState的过程中，并不会直接改变状态的值

而是把回调函数推到hook队列中，同时更新nextUnitOfWork，浏览器一旦有空闲就会重新渲染

比如调用了5个useState，在第三个useState的时候浏览器重新渲染

前两个useState是没有被打断的，那么hookIndex递增到了2

第三个的时候，又有五个useState要执行，就要从0开始了

则意味着hookIndex置为0，而且又要重新执行5个useState

由于idle的存在，可能队列中有多个action

当执行perform的时候，index初始化为0，又开始执行useState

这时wipFiber.alternate.hooks不为空，而且hookindex也有值

主要是setState和useState执行顺序的问题



栈



为了让新的hook能够和旧的hook联系起来









