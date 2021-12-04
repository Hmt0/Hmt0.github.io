---
title: 为什么只能在React函数的最顶层调用Hook?
date: 2021-10-29 17:19:55
tags: react
---

Hook是React函数组件的机制，以Counter组件为例，我们先看一下函数组件和普通组件有什么区别？

```javascript
function Counter() {
    const [state0, setState0] = Didact.useState(0)
    const [satte1, setState1] = Didact.useState(1)
    const [satte2, setState2] = Didact.useState(2)
    return (
        <h1 onClick={() => {
            setState0(c => c+1)
            setCount1(c => c+1)}}> 
            Count: {state0}
        </h1>
    )
}
```

1. 首先它是通过函数定义的，实际要渲染的内容写在返回值中
2. 在函数中可以写各种各样的Hook（`useState`）

那么为什么这种函数叫Hook呢？我们来看一下`useState`的内部原理：

在我们调用`useState`时，就会创建一个hook对象，里面包含state的值和一个actions队列。

在Counter第一次挂载的时候会把通过`useState`创建好的hook一个个添加到`wipFiber`的hooks队列中，此时`wipFiber.hook`中有三个hook:

![image-20211029145820062](C:\Users\luna6\AppData\Roaming\Typora\typora-user-images\image-20211029145820062.png)

因此，在我们执行`[state, setState] = useState(0)`的时候实际上`useState`函数返回的就是`hook.state`和一个闭包函数`setState`。

这个时候actions队列中还是空空如也的，当我们点击了Counter，就会根据`onClick`的回调函数调用相应的`setState`函数，这里做的主要工作是向对应hook（`setState`是闭包函数，保存着相应`useState`函数中定义的hook）的actions queue中加入action，也就是我们传给`setState`的回调函数，同时更新`wipRoot`。

这个时候就有可能调用到函数组件的更新了，因为我们是通过调用`requestIdleCallback`去执行组件的更新任务的，所以不能保证更新的时机，有可能执行了几个`setState`之后合并起来一起更新。

更新函数式组件时，会再次依次执行`useState`函数，与初次渲染不同的是，这次会**根据`hookIndex`获取上一个Fiber**的hooks队列中保存的对应的`hook`，这就是为什么这个保存着状态和动作的对象叫hook的原因：总是要在重新渲染时把上次的状态和动作给“钩”过来，才能实现状态的更新，也是Hook函数这么叫的原因。

使用`hookIndex`就是我们把新的`useState`中的hook和上一次联系起来的手段，所以要确保函数组件每一次执行的时候，`useState`调用的顺序都是一致的！



关于倒计时：

在`setInterval`中调用`setState`，会发现倒计时只改变了一次数字就不再变化了，把`useState`的执行过程打印出来可以看到：

![image-20211031180819358](C:\Users\luna6\AppData\Roaming\Typora\typora-user-images\image-20211031180819358.png)

只有在`useState`第二次（第一次是初次渲染）被调用的时候才在hook中添加了action，而且`setState`执行了五次，添加了五次。这是因为`setState`实在`setInterval`中调用的，而`setInterval`是在第一次更新的时候的useState中被调用的。



