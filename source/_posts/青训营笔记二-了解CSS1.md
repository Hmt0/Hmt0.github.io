---
title: 青训营笔记二-了解CSS1
date: 2022-01-28 11:13:03
tags:css
---

## 一、选择器的特异度（Specificity）

**先统计个数**：

|                        | #id  | .（伪）类 | E标签 |
| ---------------------- | ---- | --------- | ----- |
| `#nav .list li a:link` | 1    | 2         | 2     |
| `.hd ul .links a`      | 0    | 2         | 2     |

**再加权比较**大小：122>22，所以第一个选择器优先

高优先级属性值覆盖低优先级

应用：在普通按钮样式的基础上设置主要按钮的样式

```html
<button class="btn">普通按钮</button>
<button class="btn primary">主要按钮</button>
<style>
    .btn {
        display: inline-block;
        padding: .36em .8em;
        margin-right: .5em;
        line-height: 1.5;
        text-align: center;
        cursor: pointer;
        border-radius: .3em;
        border: none;
        background: #e6e6e6;
        color: #333
    }

    .btn.primary {
        color: #fff;
        background: #218de6;
    }
</style>
```

## 二、继承

某些属性会自动继承其父元素的*计算值*，除非显示指定一个值。

### 1. 显式继承

```html
<style>
	* {
        box-sizing: inherit;
    }
    html {
        box-sizing: border-box;
    }
    .some-widget {
        box-sizing: content-box;
    }
</style> 
```

### 2. 初始值

- CSS中，每个属性都有一个初始值
  - background-color的初始值为transparent
  - margin-left的初始值为0

- 可以使用initial关键字显式重置为初始值
  - `background-color:initial`

## 三、CSS求值过程

DOM树+样式规则

### 1. filtering

对应用到该页面的规则用以下条件进行筛选：选择器匹配、属性有效、符合当前media等

得到**声明值Declared Values**，一个元素的某属性可能有0到多个声明值。比如：`p{font-size:16px}`和`p.text{font-size:1.2em}`

### 2. cascading

按照来源（内联或外部）、！important、选择器特异性、书写顺序等选出优先级最高的**一个**属性值。

得到**层叠值Cascaded Value**：在层叠过程中，赢得优先级比赛的那个值，比如`1.2rem`

### 3. defaulting

当层叠值为空的时候，使用继承或初始值

得到**指定值Specified Value**，经过cascading和defaulting之后，保证指定值一定不为空。

### 4.resolving

将一些相对值或者关键字转化成绝对值，比如em转为px，相对路径转为绝对路径

得到**计算值Computed Value**，一般来说就是，浏览器会在不进行实际布局的情况下，所能得到的最具体的值。比如60%。（只根据html和css无法得到具体像素值）

### 5.formatting

将计算值进一步转换，比如关键字、百分比等都转为绝对值。

得到**使用值Used Valued**，进行实际布局时使用的值，不会再有相对值或关键字。比如400.2px。

### 6.constraining

将小数像素转为整数，或根据`min-width`等的约束调整，或浏览器对字体大小约束。

## 四、布局（layout）是什么

- 确定内容的大小和位置的算法
- 依据元素、容器、兄弟节点和内容等信息来计算

### 1. 布局相关技术

- 常规流（文档流）
  - 行级
  - 块级
  - 表格布局
  - `flexBox`
  - Grid布局
- 浮动
- 绝对定位

### 2. 盒模型

#### (1) content

width

- 指定content box宽度
- 取值为**长度、百分数、auto**
- auto由浏览器根据其它属性确定
- 百分数相对于容器的content box宽度

height

- 指定content box宽度
- 取值为**长度、百分数、auto**
- auto由内容计算得来
- 百分数相对于容器的content box高度
- 容器有指定的高度时（不是auto），百分数才生效

#### (2) padding

- 指定元素四个方向的内边距
- 百分数相对于容器**宽度**

*如何实现一个自适应正方形？*

高度为0，padding-top为100%

#### (3) border

- 指定容器边框样式、粗细和颜色
- 三种属性
  - border-width
  - border-style
  - border-color
- 四个方向
  - border-top
  - border-right
  - border-bottom
  - border-left

*技巧：实现三角形*

#### (4) margin

- 指定元素四个方向的外边距
- 取值可以是**长度、百分数、auto**
- 百分数相对于容器**宽度**

使用`margin：auto`水平居中

```h)tml
<div></div>
<style>
    div {
        width: 200px;
        height: 200px;
        background: coral;
        margin-left: auto;
        margin-right: auto;
    }
</style>
```

#### (5) margin collapse

```html
<div class="a"></div>
<div class="b"></div>

<style>
    .a {
        background: lightblue;
        height: 100px;
        margin-bottom: 100px;
    }

    .b {
        background: coral;
        height: 100px;
        margin-top: 100px;
    }
</style>
```

两个`div`的间距是100px而不是叠加成200px

只在垂直方向生效

#### (6)  `box-sizing:border-box`

默认`content-box`不包含border和padding，`border-box`指定宽高的时候包含border和padding

#### (7) overflow

- visible默认值

- hidden
- scroll
- auto溢出才显示滚动条

### 3. 块级vs.行级

| 块级盒子Block Level Box      | 行盒Inline Level Box                   |
| ---------------------------- | -------------------------------------- |
| 常规流中不和其他盒子并列摆放 | 和其它行级盒子一起放在一行或拆开成多行 |
| 使用所有盒模型属性           | 盒模型中的width、height不适用          |

| 块级元素                                             | 行级元素                                                 |
| ---------------------------------------------------- | -------------------------------------------------------- |
| 生成块级盒子                                         | -生成行级盒子<br />-**内容分散在多个行盒（line box）中** |
| body、article、div、main、section、h1-6、p、ul、li等 | span、em、strong、cite、code等                           |
| `display:block`                                      | `display:inline`                                         |

#### (1)  display属性

display属性定义了1.在相同的格式上下文中，这个盒子如何与其他元素一起显示；2.这个元素中的盒子的行为。

使用布局方法如`Flexbox` (display: flex)和Grid layout (display: Grid)的容器仍然参与**块和内联**布局，因为这些方法的外部显示类型是块。

| block        | 块级盒子                                                     |
| ------------ | ------------------------------------------------------------ |
| inline       | 行级盒子                                                     |
| inline-block | 本身是行级，可以放在行盒中；可以设置宽高；作为一个整体不会被拆散成多行 |
| none         | 排版时完全被忽略                                             |

#### (2) inline-block例子

```html
<div>
    Lorem ipsum dolor sit amet, consectetur adipisicing elit. Porro laboriosam, delectus, illum
    <img src="https://assets.codepen.io/59477/cat.png" alt="cat"/>
    And <em>Inline Block</em>
</div>
<style>
    div {
        width: 10em;
        background: #411;
    }
    em {
        display: inline-block;
        width: 3em;
        background: #33c;
    }
</style>
```

### 4.常规流Normal Flow

- 根元素、浮动和绝对定位的元素会脱离常规流
- 其他元素都在常规流之内（in-flow）
- 常规流中的盒子，在某种**排版上下文**中参与布局
  - 行级排版上下文
  - 块级排版上下文
  - Table排版上下文
  - Flex排版上下文
  - Grid排版上下文

#### (1) 行级排版上下文

- Inline Formatting Context（IFC）
- **只包含行级盒子**的容器会创建一个IFC
- IFC内的排版规则
  - 盒子在一行内水平摆放
  - 一行放不下时，换行显示
  - text-align决定一行内盒子的水平对齐
  - vertical-align决定一个盒子在行内的垂直对齐
  - 避开浮动（float）元素

上面的例子中，div内部只包含行级元素，因此创建了一个IFC

```html
<div>
    Lorem ipsum dolor sit amet, consectetur adipisicing elit. Porro laboriosam, delectus, illum
    <p>块级盒子</p>
    <img src="https://assets.codepen.io/59477/cat.png" alt="cat"/>
    And <em>Inline Block</em>
</div>
```

这段代码中，p元素上面部分是一个IFC，下面是一个IFC，p是块级盒子，里面还是IFC。

#### (2) 块级排版上下文

- Block Formatting Context（BFC）
- 某些容器会创建一个BFC
  - 根元素
  - 浮动、绝对定位、inline-block
  - Flex子项和Grid子项
  - overflow值不是visible的块盒
  - display：flow-root；

### 5. BFC内的排版规则

- 盒子从上到下摆放
- 垂直margin合并
- BFC内盒子的margin不会与外面的合并
- BFC不会和浮动元素重叠

```
<span>
    This is a text and
    <div>block</div>
    and other text.
</span>

<style>
    span {
        line-height: 3;
        border: 2px solid red;
        background: coral;
    }

    div {
        line-height: 1.5;
        background: lime;
    }
</style>
```

![image-20220129232559701](C:\Users\luna6\AppData\Roaming\Typora\typora-user-images\image-20220129232559701.png)

span内部橙色部分匿名块级盒子 行级上下文

绿色也是一个行级上下文

？？？

### 6. Flex Box是什么？

- 一种新的排版上下文

- 它可以控制子级盒子的：
  - 摆放的流向（左右上下）flex-direction
  - 摆放顺序 order
  - 盒子宽度和高度
  - 水平和垂直方向的对齐 justify-content align-items
    - 单独元素 align-self
  - 是否允许折行

#### (1) Flexibility

- 可以设置子项的弹性：当容器有剩余空间时，会伸展；容器空间不够时，会收缩。
- flex-grow：有**剩余空间**时的伸展能力
- flex-shrink：容器空间不足时收缩的能力
- flex-basis：没有伸展或收缩时基础长度

#### (2) flex-shrink默认值是1

```html
<div>
    <div class="container">
        <div class="a">A</div>
        <div class="b">B</div>
        <div class="c">C</div>
    </div>
    <style>
        .container {
            display: flex;
            width: 1000px;
            height: 500px;
        }
        .a, .b, .c {
            width: 400px;
        }

        /* a不压缩,b和c默认flex-shrink: 1 */
        .a {
            flex-shrink: 0;
            background: burlywood;
        }
        .b {
            flex-shrink: 2;
            background: brown;
        }
        .c {
            flex-shrink: 1;
            background: cadetblue;
        }
    </style>
</div>
```

#### (3) 缩写

| flex: 1         | flex-grow: 1                                    |
| --------------- | ----------------------------------------------- |
| flex: 100px     | flex-basis: 100px                               |
| flex: 2 1       | flex-grow: 2; flex-shrink: 1                    |
| flex: 1 100px   | flex-grow: 1; flex-basis: 100px                 |
| flex: 2 0 100px | flex-grow: 2; flex-shrink: 0; flex-basis: 100px |
| flex: auto      | flex: 1 1 auto                                  |
| flex: none      | flex: 0 0 auto                                  |

### 7. Grid布局display:grid

- display: grid使元素生成一个块级的Grid容器
- 使用grid-template相关属性将容器划分为网格
- 设置每一个子项占哪些行/列

#### (1) 划分网格

```css
.container {
    display: grid;
    grid-template-columns: 100px 100px 200px;
    grid-template-rows: 100px 100px;
}
```

```
.container {
    display: grid;
    grid-template-columns: 30% 30% auto;
    grid-template-rows: 100px auto;
}
```

fr：fraction

```
.container {
    display: grid;
    grid-template-columns: 100px 1fr 1fr;
    grid-template-rows: 100px 1fr;
}
```

#### (2) grid line网格线

从1开始计数

#### (3) grid area网格区域

三列三行：

```css
.container {
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
}
.a {
    background: cadetblue;
    grid-row-start: 1;
    grid-column-start: 1;
    grid-row-end: 3;
    grid-column-end: 3;
}
```

简写：

```css
.a {
	background: cadetblue;
	grid-area: 1/1/3/3;
}
```

可以重叠：

```css
.a {
    background: cadetblue;
    grid-area: 2/2/4/4;
}
.b {
    background: chocolate;
    grid-area: 1/1/3/3;
}
```

### 8. float浮动

背景：图文环绕

其他情况不用

### 9. position属性

- static 默认值，非定位元素
- relative 相对自身原本位置偏移，不脱离文档流
- absolute 绝对定位。相对非static祖先元素定位
- fixed 相对于视口绝对定位

#### (1) position: relative

- 在常规流里面布局
- 相对于自己本应该在的位置进行偏移
- 使用top、left、bottom、right设置便宜长度
- 流内其它元素当他没有偏移一样布局

#### (2) position: absolute

- 脱离常规流
- 相对于**最近的非static祖先**定位
- 不会对流内元素布局造成影响

