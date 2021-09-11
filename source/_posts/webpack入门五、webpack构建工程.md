---
title: webpack入门五、webpack构建工程
date: 2021-09-10 22:05:39
tags: webpack
---

# `webpack`构建工程

#### 学习目标

- 使用`webpack`构建真实`react`工程
- 掌握`babel`的用法，理解`babel`原理
- 掌握高频`loader`和`plugin`的用法
- 掌握生产级别的`webpack`配置方法

#### 准备工作：

```powershell
mkdir learn-webpack
cd learn-webpack
npm init
npm init -y // 把所有预置项设置为npm默认
ls
npm install react react-dom
npm install webpack webpack-cli -d
```

#### 项目文件：

`App.jsx`:

```javascript
import React from 'react'
import ReactDom from 'react-dom'
import { isNull, isZero } from './utils'

isNull({})

const App = () => {
    return (
        <div>
            <h1>webpack学习</h1>
        </div>
    )
}

export default App
ReactDom.render(<App/>, document.getElementById('app'))
```

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
</body>
</html>
```

`webpack`读不懂`ES6`语法，也无法处理`html`文件。

#### babel：

把高版本代码编译成低版本代码

```
npm install @babel/core @babel/cli -g
```

```huozhe
babel test.js --presets=@babel/preset-env
```

`--presets=@babel/preset-env`是配置参数，指定转换规则。`presets-env`可以把高版本代码转换为低版本。写在配置文件中：

`package.json`或者`.babelrc`文件(优先查询)

```javascript
"babel": {
	"presets": ["@babel/presets-env"]
}
```

#### 在`webpack`中使用`babel`

`webpack.config.js`:

```javascript
module.exports = {
    module: {
        rules: [
            {
                test: /\.jsx?/,
                exclude: /node_modules/, // 不对node_module进行编译
            }
        ]
    }
}
```

安装loader和规则：解决代码转译问题：

```
npm install babel-loader
npm install @babel/preset-env @babel/preset-react
```

填写规则：

```javascript
options: {
    babelrc: false,
    presets: [
        require.resolve('@babel/preset-react'),
        // 转义jsx
        [require.resolve('@babel/preset-env', {modules: false})]
        // 转义ES6，webpack可以识别import语法
    ],
    cacheDirectory: true, // 是否对转译结果做缓存
}
```

#### `html`文件处理：`plugin`

`plugin`实现节点层面的处理，以构造函数的形式存在

```
npm install html-webpack-plugin -d
```

```javascript
plugins: [
        new HtmlWebPackPlugin({
            template: path.join(__dirname, 'src/index.html'),
            // 需要被处理的文件的路径
            filename: 'index.html'
        })
    ]
```

入口文件`index.jsx`

```
import App from './App'
```

import省略文件后缀：

```javascript
 resolve: {
        extensions: ['.wasm', '.mjs', '.js', '.jsx', 'json']
 },
     entry: path.resolve(__dirname, 'src/index.jsx'), // 入口文件
```

> ERROR in ./src/index.html 1:0
> Module parse failed: Unexpected token (1:0)
> You may need an appropriate loader to handle this file type, currently no loaders are configured to process this file. See https://webpack.js.org/concepts#loaders
>
> > <!DOCTYPE html>
> > | <html lang="en">
> > | <head>
>
> webpack 5.52.0 compiled with 1 error in 93 ms

babel与babel-loader版本应该相匹配https://www.npmjs.com/package/babel-loader

#### 打包

```powershell
webpack-dev-server --config // 指定读取的config文件,比如webpack.config.dev.js
webpack-dev-server --open // 打开浏览器
```

自定义命令：

```javascript
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack --mode production", // 打包
    "start": "webpack-dev-server --mode development --open" // 预览
  },
```

`HMR`热更新（热替换）， 在不刷新页面的情况下更新修改部分：

```javascript
devServer: {
	hot: true
}
Plugins: [
    ...
    new webpack.HotModuleReplacementPlugin()
    ...
]
```

`index.jsx`

```javascript
if(module.hot) {
    module.hot.accept(error => {
        if(error) {
            console.log("热替换出错")
        }
    })
}
```

