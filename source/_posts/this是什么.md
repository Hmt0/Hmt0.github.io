---
title: this是什么
date: 2021-09-19 21:14:26
tags: JavaScript
---

# this是什么？

this是函数内部的一个对象，引用的是把函数当成**方法**的**上下文**对象。

- 在标准函数中，this引用的是把函数当成方法**调用**的上下文对象。

- 在箭头函数中，this引用的是**定义**箭头函数的上下文。

那么上下文对象是什么呢？

上下文是**执行**上下文的简称，主要有全局上下文和局部上下文两种，每个函数调用都有自己的上下文。变量或函数的上下文决定了它们可以访问哪些数据，以及他们的行为。每个上下文都有一个关联的**变量对象**，而这个上下文中定义的所有变量和函数都存在于这个对象上。

我的理解呢就是，作用域链的数据结构类似于栈，每个函数都有自己的作用域链，**定义**时初始化，预装载全局变量。函数**调用**时，创建执行上下文，更新作用域链。

先来看一下标准函数，我们常写的是下面这种方式，A是全局上下文中的对象，`sayHello`是作为A的方法调用的，因此`this`引用的是A：

```javascript
var A = {
    name: 'A',
    sayHello: function(){
       console.log(this.name)
    }
}

A.sayHello() // A
```

那么如果在A中执行呢？因为A定义是在全局作用域中，所以其实和直接在全局执行一样，立即执行函数的this引用的是Window：

```javascript
var A = {
    name: 'A',
    sayHello: (function(){
       console.log(this) // Window
    })()
 }
```

这样就不难理解箭头函数中this的引用了：

```javascript
var A = {
    name: 'A',
    sayHello: () => {
       console.log(this.name)
    }
}

A.sayHello() // Cannot read property 'name' of undefined
```

在标准函数中this和arguments是在被调用时创建的，在闭包中使用this时，内部函数永远不可能直接访问外部函数的this和arguments变量：

```javascript
var A = {
    name: 'A',
    sayHello: function(){
        return function(){
            console.log(this)
        }
    }
}

A.sayHello()() // Window
```

可以通过把this保存到闭包可以访问的另一个变量来解决这个问题。下面的例子中，箭头函数是在标准函数中定义并返回的，this引用的是`A.sayHello`的上下文:

```javascript
var A = {
    name: 'A',
    sayHello: function(){
        return () => {
            console.log(this.name)
        }
    }
}

A.sayHello()() // A
```

在函数定义阶段呢，就是Window：

```javascript
var A = {
    name: 'A',
    sayHello: (function() {
        (() => {
            console.log(this) // Window
        })()
    })()
}
```

