---
title: 'webpack入门二、模块化方案进化史'
date: 2021-09-07 16:53:08
tags: webpack
---

# 模块化方案进化史

 `AMD`（Asynchronous Module Definition）异步模块定义

```javascript
// 求和模块的定义
define('getSum', ['math'], function(math)){
    return function(a, b) {
    	console.log("sum: " + math.sum(a, b));
	}
})
```

`COMMONJS`

```javascript
// 通过require函数来引入
const math = require('/math');

// 通过exports将其导出
exports.getSum = function(a, b) {
	return a + b;
}
```

`ES6 MODULE`

```javascript
// import导出
import math from './math'

// export导出
export function sum(a, b) {
    return a + b;
}
```

# `webpack`的打包机制

#### 学习目标

- `webpack`与立即执行函数之间的关系
- `webpack`打包的核心逻辑

#### 项目文件内容

`index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>React Webpack测试</title>
</head>
<body>
    <div id="app"></div>
    <script src="./src/index.js"></script>
</body>
</html>
```

`/src/index.js`

```javascript
function test() {
	console.log('我是一个小测试')
}

test()
```

#### 唤起`webpack`

```shell
webpack
```

`webpack`默认查找`/src/index.js`，打包`index.js`的代码逻辑，目标文件：`/dist/main.js`。此时，`index.html`中引入`/dist/main.js`的效果是一样的。

`main.js`中为了压缩文件体积，变量名用尽可能少的字母表示，打包结果还是立即执行函数。

引入`loadash`:

```javascript
import _ form loadash
function test() {
	console.log('我是一个小测试')
}

test()
```

此时打包结果中立即执行函数一个入参是`test函数`，另一个是`lodash`的逻辑，都作为依赖模块放在modules*数组*中作为参数，等待代码运行时调用。

大体结构：

```javascript
(function(modules) {
    var installedModules = {}; // 已经加载的模块
    
    function __webpack_require__(moduleId) { // 浏览器环境下的require
        /* code */
    }
    // ...
    return __webpack_require__(0); // entry file加载工程入口模块
}){[ /* modules array */]}
```

核心方法：

```javascript
function __webpack_require__(moduleId) {
	// Check if module is in cache
	if(installedModules[moduleId]) {
		return installedModules[moduleId].exports;
	}
    // Create a new module {and put it into the cache}
    var module = installesModules[moduleId] = {
        i: moduleId,
        l: false,
        exports: {}
    }
    // Execute the module function
    modules[moduleId].call(
    	module.exports,
        module, module.export,
        __webpack_requrie__
    );
    // Flag the module as loaded
    module.l = true;
    Return module.exports;
}
```

`webpack`打包过程：

- 从入口文件开始，分析整个应用的*依赖树*，
- 将每个依赖模块包装起来，放到一个数组中等待调用
- 实现模块加载的方法，并把他放到模块执行的环境中，确保模块间可以互相调用(__webpack_require__)
- 把执行入口文件的逻辑放在一个函数表达式中，并立即执行这个函数