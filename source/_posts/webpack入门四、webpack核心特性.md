---
title: webpack入门四、webpack核心特性
date: 2021-09-10 21:28:46
tags: webpack
---

#  webpack核心特性

#### 学习目标

- 使用webpack构建简单工程
- 了解webpack配置文件
- 掌握“一切皆模块与loaders”的思想
- 理解webpack中的“关键人物”---入口文件、出口文件、loaders、plugins

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
    <div src='/dist/bunle.js'></div>
</body>
</html>
```

`moduleLog.js`

```javascript
export default function() {
	document.write('moduleLog is loaded')
}
```

`src/index.js`

```javascript
import moduleLog from '../moduleLog'
document.write('入口js is loaded');
moduleLog();
```

#### 安装webpack和webpack-cli:

```
npm install webpack
npm install webpack-cli -g
```

> 报错：无法将“WEBPACK”项识别为 CMDLET、函数、脚本文件或可运行程序的名称

出现这个错误的原因是：windows10中，你的webpack是局部安装的，并没有加入到系统环境变量中，所以控制台找不到webpack命令，有以下方法可以解决：

- 找到项目根目录下的**package.json**文件，配置`'scripts'`这个选项，配置加上`"build"："webpack"`，然后运行`npm run build`
- 全局安装webpack

默认入口文件：src/index.js 

出口：dist/main.js

#### 自定义配置文件：

`webpack.config.js`

```javascript
const path = require('path');
module.exports = {
    entry: './app.js', // 修改入口文件
    output: {
        path: path.join(__dirname, 'dist'),
        filename: 'bundle.js'
}
}
```

> 警告：WARNING in configuration
> The 'mode' option has not been set, webpack will fallback to 'production' 
> for this value.
> Set 'mode' option to 'development' or 'production' to enable defaults for 
> each environment.

```
mode: 'development'
```

但是每次修改都要保存、打包很麻烦，有没有自动的方法呢？

#### 自动打包工具：webpack-dev-server

启动`webpack-dev-server`可以监听工程目录文件的改动，修改源文件并再次保存可以实现动态实时打包，自动刷新浏览器。

不管cli还是webpack-dev-server都是可执行文件，一种方法是全局安装，或者进入node_module/.bin/server执行

> 报错：.\webpack-dev-server : 无法加载文件 C:\Users\hh\github\learnwebpack\node_
> modules\.bin\webpack-dev-server.ps1，因为在此系统上禁止运行脚本。有关详细 
> 信息，请参阅 https:/go.microsoft.com/fwlink/?LinkID=135170 中的 about_Exe 
> cution_Policies。

1. 以管理员身份运行powershell
2. 输入set-executionpolicy remotesigned，回车
3. 输入yes

指定本地服务端口：

```javascript
devServer: {
	port: 3000, // 服务端口
	publicPath: '/dist'
}
```

此时删除bundle.js依然可以运行，说明只在内存中生成了一个bundle.js，当浏览器发出请求时才会运行。

所有资源（图片、css等）都是模块

#### 文件编译：loader

安装css loader和style loader

```powershell
npm install css-loader --save-dev
npm install style-loader --save-dev
```

loader本质上是文件加载器

```javascript
 {
     test: /\.css$/,
         use: [
             'style-loader', // 加载样式
             'css-loader' // 编译css文件，注意配置顺序与加载顺序相反
         ]
 }
```

#### plugins：

监听事件，改变文件打包输出结果，比如压缩资源体积，去掉代码中的注释。

举例：引入uglify之前的打包结果：`asset bundle.js 4.26 KiB [emitted] (name: main) `

安装：

```
npm install uglifyjs-webpack-plugin --save-dev
```

引入：

```javascript
const UglifyJSPlugin = require("uglifyjs-webpack-plugin")
...
Plugins: [
	new UglifyJSPlugin()
]
...
```

`asset bundle.js 1.82 KiB [emitted] (name: main)`

代码体积减小了。
