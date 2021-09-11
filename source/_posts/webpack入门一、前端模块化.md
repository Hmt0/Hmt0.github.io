---
title: 'webpack入门一、前端模块化'
date: 2021-09-07 16:53:08
tags: webpack
---

# webpack是什么

> webpack 是一个用于现代 JavaScript 应用程序的 *静态模块打包工具*。当 webpack 处理应用程序时，它会在内部从一个或多个入口点构建一个 [依赖图(dependency graph)](https://webpack.docschina.org/concepts/dependency-graph/)，然后将你项目中所需的每一个模块组合成一个或多个 *bundles*，它们均为静态资源，用于展示你的内容。

#### 1、首先，为什么需要打包？

逻辑多、文件多，项目复杂度提高了。

#### 2、在打包之外，webpack还有哪些功能和特点？

`loader`：用于对模块的源代码进行转换。

`plugin`：打包优化，资源管理，注入环境变量。

`loader`和`plugin`的特点：可插拔 =>webpack不仅强大，而且灵活

#### 3、 课程重点：

- 理解前端模块化
- 理解wenpack打包的核心思路
- 理解webpack中的“关键人物”---`loader`和`plugin`

# 理解前端模块化

- 作用域
- 命名空间
- 模块化

#### 1、作用域

作用域描述了代码*运行时*变量、函数、对象的*可访问性*

作用域决定了代码中变量和其他资源的可见性，JavaScript有全局作用域和局部作用域之分。全局作用域中的变量和资源都会挂载在全局对象，在浏览器中是window，在node中是global。以下是在全局作用域中声明变量和局部作用域的区别：

```javascript
var a = 1
window.a // 1

function test() {
	var b = 2
}

b // Uncaught ReferenceError

```

#### 2、模块化的演化过程

过去当一个页面想要组合多个功能时，使用引入多个JavaScript外链文件的方式：

```javascript
<script src="./moduleA.js"></script>
<script src="./moduleB.js"></script>
<script src="./moduleC.js"></script>
```

缺陷：moduleA、moduleB、moduleC共用一个全局作用域，在任何一个JavaScript文件中进行顶层的变量声明，都会暴露在全局中，使得其他脚本会获得它并不想要的变量。当应该规模和复杂度上升，这些脚本之间可能会发生*命名冲突*：

解决方法：给不同文件不同的*命名空间*

```javascript
// ModuleA
var a = {
	name: 'Susan'
}

// ModuleB
var b = {
    name = 'Jack'
}

// ModuleC
console.log('moduleA里的name: ', a.name) //moduleA里的name: Susan
```

新的缺陷：其他文件可以轻易修改命名空间中对象的属性。无法保护对象属性被随意访问和篡改：

```javascript
// ModuleA
var Susan = {
	name: 'Susan',
	sex: '女孩',
	// 自我介绍方法
	tell: function() {
		console.log('我的名字是', this.name)
		console.log('我的性别是', this.sex)
	}
}

// ModuleB
var Jack = {
	name: 'Jack',
	sex: '男孩',
	// 自我介绍方法
	tell: function() {
		console.log('我的名字是', this.name)
		console.log('我的性别是', this.sex)
	}
}
```

```javascript
Susan.name = 'Jack'
Susan.name // Jack
```

举个例子，在自动登录功能中，对于用户账号密码这样的敏感信息，不希望被其他模块获取、修改：

```javascript
// ModuleA
autoLogin={
	userName="小明",
    password: "123456",
    login: function() {
        // 利用useName和password登录
    }
}
```

```javascript
// ModuleB
autoLogin.password = "32"
```

解决方法：利用函数作用域保护模块中的变量，形成模块作用域，隐藏该隐藏的，暴露该暴露的。实现一个立即执行函数，并返回*闭包函数*：

```javascript
var Susanodule = (function(){
    var name = 'Susan'
    var sex = '女孩'
    return {
        tell: function() {
            console.log('我的名字是：', name)
            console.log('我的性别是：', sex)
        }
    }
})()

SusanModule.tell // 我的名字是我的性别是
SusanModule.name // undefined
SusanModule.sex // undefined
```

这样暴露了一个tell方法，但是其他模块无法直接访问修改name和sex的值。

改进：在满足重用的基础上，将赋值过程写在立即执行函数内部

```javascript
(function(window){
    var name = 'Susan'
    var sex = '女孩'
    function tell(){
        console.log('我的名字是：', name)
        console.log('我的性别是：', sex)
    }
    window.SusanModule = {tell}
})(window)
```

```javascript
SusanModule // {tell: f}
```

总结一下，模块化的优点：

- 作用域封装 - 用模块把代码封装起来，保证模块内部的实现不会暴露在全局作用域中，只需要将模块的功能通过接口的方式暴露出去给其他模块调用，避免污染全局命名空间
- 重用性
- 接触耦合 - 模块间接口不变，模块内部的变化对其他模块没有影响

