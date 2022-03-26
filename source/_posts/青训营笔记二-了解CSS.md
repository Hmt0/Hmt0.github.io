---
title: 青训营笔记二-了解CSS0
date: 2022-01-24 14:24:13
tags: css
---

## 一、CSS概述

### 1. CSS是什么

Cascading Style Sheets

- 用来定义页面元素的样式

  - 设置字体和颜色

  - 设置位置和大小

  - 添加动画效果

### 2. 组成

1. 选择器Selector：选中页面中的元素
2. 声明Declaration
  - 属性Property
  - 属性值Value

### 3. 在页面中使用CSS

```html
<!-- 外链 -->
<link rel="stylesheet" href="/assets/style.css"/>

<!-- 嵌入 -->
<style>
    li {margin: 0; list-style: none;}
    p {margin: lem 0;}
</style>

<!-- 内联 -->
<p style="margin: 1em 0">Example Content</p>
```

### 4. CSS是如何工作的

一个基础过程：

![image-20220117110925287](C:\Users\luna6\AppData\Roaming\Typora\typora-user-images\image-20220117110925287.png)

## 二、选择器Selector

- 找出页面中的元素，以便给他们设置样式
- 使用多种方式选择元素
  - 按照标签名、类名或id
  - 按照属性
  - 按照DOM树中的位置

### 1. 通配选择器

匹配所有标签元素

```css
<style>
* {
    color: red;
    font-size: 20px;
}
</style>
```

### 2. id选择器

元素的id属性的值是唯一的

```css
#logo {
    font-size: 60px;
    font-weight: 200;
}
```

### 3. 类选择器

选择同一类型的元素

```css
.done {
	color: gray;
	text-align: line-through;
}
```

### 4. 属性选择器

选择具有这个属性的元素

```html
<label>用户名：</label>
<input value="zhao" disabled/>

<label>密码：</label>
<input value="123456" type="password"/>

<style>
    [disabled] {
        background: #eee;
        color: #999;
        }
</style>
```

选择属性是某个具体值的元素

```CSS
input[type="password"] {
    border-color: red;
    color: red;
}
```

### 5. 匹配

```css
/*以“#”开头*/
a[href^="#"] {
	color: red;
}
/* 以“.jpg”结尾 */
a[href$=".jpg"] {
	color: deepskyblue;
}
```

## 三、伪类（pseudo-classes）

- 不基于标签和属性定位元素
- 几种伪类
  - 状态伪类
  - 结构性伪类

### 1. 状态性

```html
<a href="http://example.com">
    example.com
</a>

<label>
    用户名：
    <input type="text"/>
</label>

<style>
    /* 默认 */
    a:link {
        color: black;
    }
    /* 已访问 */
    a:visited {
        color: gray;
    }
    /* 鼠标停留 */
    a:hover {
        color:orange;
    }
    /* 链接鼠标按下时 */
    a:active {
        color: red;
    }
    /* 鼠标按下时 */
    :focus {
        outline: 2px solid orange;
    }
</style>
```

### 2. 结构伪类

根据元素出现在父级节点的相对位置选择

```html
<ol>
    <li>🍎</li>
    <li>🍎</li>
    <li>🍎</li>
    <li>🍎</li>
    <li>🍎</li>
</ol>
<style>
    li {
        list-style-position: inside;
        border-bottom: 1px solid;
        padding: 0.5em;
    }

    li:first-child {
    	color: coral;
    }
    li:last-child {
    	border-bottom: none;
    }
</style>
```

![image-20220117143601170](C:\Users\luna6\AppData\Roaming\Typora\typora-user-images\image-20220117143601170.png)

## 四、组合

### 1. 直接组合

```html
<label>
    用户名：
    <input class="error" value="aa"/>
</label>
<span class="error">
    最少三个字符
</span>
<style>
    .error {
        color: red;
    }
    input.error {
        border-color: red;
    }
</style>
```

| 名称       | 语法 | 说明                        | 示例        |
| ---------- | ---- | --------------------------- | ----------- |
| 直接组合   | AB   | 同时满足A和B                | input:focus |
| 后代组合   | A B  | 选中B，如果它是A的子孙      | nav a       |
| 亲子组合   | A>B  | 选中B，如果它是A的子元素    | section>p   |
| 兄弟选择器 | A~B  | 选中B，如果它在A后且和A同级 | h2~p        |
| 相邻选择器 | A+B  | 选中B，如果它紧跟在A后面    | h2+p        |

### 2. 选择器组

```css
body, h1, h2, h3, h4, h5, h6, ul, ol, li {
    margin: 0;
    padding: 0;
}
[type="checkbox"], [type="radio"] {
    box-sizing: border-box;
    padding: 0;
}
```

## 五、颜色-RGB

- rgb(0, 0, 0)

- #000000

- HSL

  - Hue 色相是颜色的基本属性，如红色、黄色等；取值是0~360

  - Saturation 饱和度是色彩的鲜艳程度；取值范围0-100%

  - Lightness 亮度是颜色的明亮程度；取值范围是0-100%

### 1. 颜色关键字

navy olive...

### 2. alpha 透明度

- #ff0000**bd**
- rgba(255, 0, 0, **0.74**)

- hsla(0, 100%, 50%, **0.74**)


## 六、文字属性

属性值为一系列字体，因为不同设备能显示的字体不同。

```html
<h1>Lorem ipsum dolor sit</h1>
<p>Lorem, ipsum dolor sit amet consectetur adipisicing elit. Ratione sit optio fugit distinctio reprehenderit nisi debitis, at similique ex architecto inventore earum deleniti. Ea velit voluptatem saepe, similique quam nisi?</p>

<style>
    h1 {
        font-family: Optima, Georgia, 'Times New Roman', Times, serif;
    }
    body {
        font-family: Arial, Helvetica, sans-serif;
    }
</style>
```

### 1. 通用字体族font-family

- Serif：衬线体 在笔画末端有装饰

- Sans-Serif：无衬线体

- Cursive：手写体

- Fantasy

- Monospace：主要针对英文，每个字符的宽度一致，如编程常用的字体


### 2. font-family使用建议

- 字体列表最后写上**通用**字体族
- 英文字体放在中文字体前面（中文字体一般包含英文字符，反之不然）

*在开发者工具中的计算样式里可以查看实际渲染的字体*

![image-20220119001300375](C:\Users\luna6\AppData\Roaming\Typora\typora-user-images\image-20220119001300375.png)

### 3. 使用Web Fonts

`@font-face`引入自定义字体

由于中文字体文件一般较大，实际开发中一般对字体文件进行裁剪，使用只包含页面中出现的文字的子包。

### 4. font-size

- 关键字
  - small、medium、large
- 长度
  - px、em
- 百分数
  - 相对父元素字体的大小

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <section>
        <h2>A web font example</h2>
        <p class="note">Notes: Web fonts...</p>
        <p>With this in mind, let's build...</p>
    </section>

    <style>
        section {
            font-size: 20px;
        }

        section h2 {
            font-size: 2em;
        }

        section .note {
            font-size: 80%;
            color: orange;
        }
    </style>
</body>
</html>
```

上面的代码中，h1的实际大小是40px，note的实际大小是16px

### 5. `font-style`

可以设置粗体、斜体（italic）等

### 6. `font-weight`字重 字体粗细

常用的两个设置：400-normal，700-bold

依赖本地安装字体实现

### 7.`line-height`

每行文字间距

```html
<body>
    <h1>Font families recap</h1>
    <p>Lorem ipsum dolor sit amet consectetur adipisicing elit. Inventore sed perspiciatis sequi quaerat officia, esse explicabo ipsum? Natus labore quis incidunt est blanditiis quidem corrupti, assumenda hic mollitia saepe at!</p>
    <style>
        h1 {
            font-size: 30px;
            line-height: 45px;
        }
        p {
            font-size: 20px;
            line-height: 1.6;
        }
    </style>
</body>
```

p元素的行高是1.6*20px

### 8. 同时设置多个属性

```
font: style weight size/height family
```

```html
<style>
    h1 {
        font: bold 14px/1.7 Helvetica, sans-serif;
    }
    p {
        font: 14px serif;
    }
</style>
```

### 9. `text-align`文字对齐方式

- left
- center
- right
- justify两端对齐 对一段文字的最后一行不生效

### 10. spacing

`letter-spacing`：字母间距

`word-spacing`：单词间距

`text-indent`：首行缩进

### 11. `text-decoration`文本修饰

- none
- underline
- line-through
- overline

### 12. white-space空白符

*代码中的回车都会保留*

- normal 默认多个连续空白符会合并，自动换行
- nowrap合并空格，不自动换行
- pre保留空格，不自动换行
- pre-wrap保留空格，自动换行
- pre-line合并空格，自动换行

# 七、调试CSS

快捷键：

Ctrl+Shift+I（Windows）

Cmd+Opt+I（Mac）

# 课后提问

优雅降级和渐进增强

css in js：css的问题：变量全局生效

