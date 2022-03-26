---
layout: 
title: 青训营笔记一 前端与HTML
date: 2022-01-18 17:37:28
tags: html
---

本节课老师梳理了HTML的相关概念，同时学了一些不常用的标签和属性。

## 一、前端概述

### 1. 技术栈

- js

- css

- html


### 2. 前端应该关注哪些方面？

图形界面下的人机交互

功能 美观 无障碍 安全 性能 兼容性 体验

### 3. 前端的边界

- `node` - 服务器端应用
- `Electron`/`React Native` - 跨平台桌面应用
- `WebRTC`（Web Real-Time Communication） - P2P在线传输，多人会议
- WebGL（Web Graphics Library） - 3D游戏

- WebAssembly - 可以使用非 JavaScript 编程语言编写代码并且能在浏览器上运行


### 4.开发环境

浏览器+编辑器

## 二、HTML概述

### 1. HTML是什么？

- HyperText（图片、链接、表格等） 

- Markup（ 标记语言，`<img src="photo.jpg" />`，`src`是属性名，等号右边是属性值）

- Language

一个完整的例子：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>页面标题</title>
</head>
<body>
    <h1>一级标题</h1>
    <p>段落内容</p>
</body>
</html>
```

`<!DOCTYPE html>`：对 HTML文件来说，浏览器使用文件开头的 DOCTYPE 来决定用**怪异模式**处理或**标准模式**处理。

`html`：文档根标签，所有其他元素必须是此元素的后代。

`head`：页面元数据，不需要直接呈现给用户

`body`：需要呈现给用户的内容

### 2. DOM树

浏览器对html文件进行解析，得到DOM树，DOM节点

## 三、HTML语法

- 标签和属性不区分大小写，推荐**小写**
- 空标签可以不闭合，比如input、meta
- 属性值推荐用双引号包裹
- 某些属性值可以省略，比如 required、readonly

### 1.标题h1~h6

### 2.列表

- 有序列表`<ol>`：ordered list
- 无序列表`<ul>`：unordered list
- key-value列表`<dl>`：defination list
  - `<dt>`title
  - `<dd>`description
  - 多对多的关系


### 3.链接anchor

```
<a href="www.aboutmt.top"></a>
```

- `target`：该属性指定在何处显示链接的资源。 取值为标签（tab），窗口（window），或框架（iframe）等浏览上下文的名称或其他关键词。

### 4. 多媒体：图片和音视频

`<img/>`

- `alt`：定义了图像的备用文本描述。

`<audio />`

`<video />`

- `controls`：默认播放控件

### 5.输入

```html
<input placeholder="range"/>
<input type="range"/>
<input type="number" min="1" max="10"/>
<input type="date" min="2018-02-10"/>
<textarea>Hey</textarea>
```

### 6.选择

多选：

```html
 <p>
     <label><input type="checkbox" />🍎</label>
     <label><input type="checkbox" />🍌</label>
</p>
```

单选：通过name达到互斥关系

```html
<p>
    <label>
        <input type="radio" name="fruit" />🍎
    </label>
        <label><input type="radio" name="fruit" />🍌		</label>
</p>
```

下拉菜单：

```html
<p>
    <select>
        <option>🍎</option>
        <option>🍌</option>
        <option>🍈</option>
    </select>
</p>
```

自由输入的同时有提示：

```html
<input list="fruits" />
<datalist id="fruits">
    <option>🍎</option>
    <option>🍌</option>
    <option>🍈</option>
</datalist>
```

### 7.文本类标签

#### 文字引用

```html
<blockquote cite="http://aboutmt.top">
    长引用，直接引用别人的一段文字，可以换行
</blockquote>
```

```html
<p>
    短引用，我最喜欢的一本书是<cite>小王子</cite>
    引用内容不要包含换行
</p>
```

```html
<p>
    在<cite>第一章</cite>，我们讲过<q>字符串是
    不可变量。</q>
</p>
```

`q`和`cite`区别：

- `cite`表示一个作品的引用，且必须包含作品的标题；

- `q`表示一个封闭的并且是短的行内引用的文本。

#### 代码引用：

使用等宽字体（编程字体）展示

```html
<p>
    <code>const</code>声明创建一个只读的常量。
</p>
```

```html
<pre>
	<code>
    	const add = (a, b) => a + b;
        const multiply = (a, b) => a * b;
    </code>
</pre>
```

#### 强调标签

```html
<p>
    在投资之前，<strong>一定要做风险评估</strong>。
</p>
<p>Cats <em>are</em> cute animals.</p>
```

- `strong`强调内容重要
- `em`强调语气

### 8. 内容划分

`header`

`nav`

`main`

`article`

`aside` 广告 热点推荐

`footer` 版权信息 备案

## 四、语义化是什么

- HTML中的元素、属性及属性值都拥有某些含义

- 开发者应该遵循语义来编写HTML

  - 有序列表用ol；无序列表用ul

  - lang属性表示内容所使用的语言

有助于chrome自动翻译

### 1. 谁在使用我们写的HTML

- 开发者-修改、维护页面
- 浏览器-展示页面，比如语言
- 搜索引擎-提取关键词、排序
- 屏幕阅读器-给盲人或者不方便的人页面内容 无障碍性

### 2. 语义化的好处

- 代码可读性
- 可维护性
- 搜索引擎优化
- 提升无障碍性

### 3. 传达内容，而不是样式

```html
<p style="font-size: 32px;">
	前端工程师的自我修养（这是一个不好的例子）
</p>
<h1>前端工程师的自我修养（推荐这样写）</h1>
```

### 4. 如何做到语义化？

- 了解每个标签和属性的含义
- 思考什么标签最适合描述这个内容
- 不使用可视化工具生成代码

