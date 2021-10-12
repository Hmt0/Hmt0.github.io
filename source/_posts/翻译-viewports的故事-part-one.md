---
title: '翻译:viewports的故事-第一部分'
date: 2021-09-23 15:11:28
tags: html,翻译
---



原文地址：https://www.quirksmode.org/mobile/viewports.html

以下为翻译：

**在这个短系列中我将阐述`viewports`和一些重要的元素的widths属性是如何工作的，比如<html>标签、窗口和屏幕。**

这一篇是关于桌面浏览器的，它的唯一目的是为移动浏览器的类似讨论奠定基础。许多web开发者已经对许多有关桌面的概念有了直观的理解。在手机端我们将会看到相同的概念，只不过更加复杂，一个关于大家都知道的术语的先导讨论将会极大帮助地你理解手机浏览器。

### 概念：设备像素和`css`像素

第一个需要理解的概念是`CSS`像素以及它和设备像素的区别。

设备像素就是我们直觉上认为“正确”的像素。这些像素就是你正在使用的设备的正式分辨率，通常可以通过`screen.width/height`读取。

如果你给某个元素设置`width: 128px`，而你的显示器的宽度是`1024px`，并且你把浏览器窗口最大化，那么这个元素可以在你的显示器上填充8次（粗略地;让我们忽略棘手的部分）。

然而，如果用户缩放，这个计算关系将会改变。如果用户放大至200%，你的`width: 128px`的元素将只能在`1024px`的显示器填充4次。

在现代浏览器中实现的缩放只不过是“扩展”像素。也就是说，不是元素的宽度从128变为256像素；而是*实际像素*的大小翻倍。形式上，元素的宽度仍然是128 ·`CSS`像素，即使它恰好占用256个设备像素的空间。

换句话说，放大200%使`CSS`像素扩大为设备像素的四倍。（宽度两倍，高度两倍，总共四倍）。

一些图片可以明确这些概念。这是在100%缩放级别的四个像素，`CSS`像素与设备像素完全重叠。

![img](https://www.quirksmode.org/mobile/pix/viewport/csspixels_100.gif)

现在让我们缩小。`CSS`像素开始收缩，这意味着一个设备像素现在重叠了几个`CSS`像素。

![img](https://www.quirksmode.org/mobile/pix/viewport/csspixels_out.gif)

如果你放大，会发生相反的情况。`CSS`像素开始增长，现在一个`CSS`像素重叠了几个设备像素。

关键点在于*我们只关心`CSS`像素*。正是这些像素决定了样式表的呈现方式。

设备像素几乎没用。不是对用户；用户会放大或缩小页面，直到可以舒服地阅读它。然而，缩放级别对你来说并不重要。浏览器会自动确保你的`CSS`布局被拉伸或压缩。

### 100%缩放

我一开始用假设缩放级别为100%举例。现在是时候更严格地定义它了：

> 在缩放级别100%，一个`CSS`像素完全等于一个设备像素。

100%缩放的概念在接下来的说明中非常有用，但在日常工作中你不应该过度关注它。在桌面，你通常会用100%的缩放来测试你的网站，但即使用户放大或缩小，神奇的`CSS`像素也会确保你的布局保持相同的比例。

### 屏幕尺寸 Screen size

让我们看看一些实际的测量方法。我们从`screen.width`和`screen.height`开始。它们包含用户屏幕的总宽度和高度。这些尺寸用设备像素来衡量，因为它们永远不会改变：它们是显示器的特性，而不是浏览器的特性。

![img](https://www.quirksmode.org/mobile/pix/viewport/desktop_screen.jpg)

有趣！但是我们要怎么处理这些信息呢？

基本上没什么用。用户显示器的大小对我们来说并不重要，除非你想在web统计数据库中使用它。

### 窗口尺寸 Window size

相反，我们想知道的是浏览器窗口的内部尺寸。它会告诉你用户当前有多少空间用来展示你的`CSS`布局。你可以通过`Window.innerWidth`和`window.innerHeight`来查询这些尺寸。

![img](https://www.quirksmode.org/mobile/pix/viewport/desktop_inner.jpg)

显然，窗口的内部宽度是以`CSS`像素衡量的。你需要知道你的布局有多少可以挤进浏览器窗口，并且这个数量会随着用户的放大而减少。如果用户放大你会得到更少的可用空间，`innerWidth/Height`的递减可以反映这一点。

(Opera是个例外，当用户放大时，它的`innerWidth/Height`不会减少：它们是以设备像素衡量的。这在台式机上很烦人，但在移动设备上却是致命的，我们稍后会看到。)

![img](https://www.quirksmode.org/mobile/pix/viewport/desktop_inner_zoomed.jpg)

注意，测量的宽度和高度包括滚动条。它们也被认为是内窗的一部分。(这主要是由于历史原因。)

### 滚动抵消 Scrolling offset

`window.pageXOffset`和`window.pageYOffset`包含文档的水平和垂直滚动偏移量。这样你就可以知道用户滚动了多少。

![img](https://www.quirksmode.org/mobile/pix/viewport/desktop_page.jpg)

这些属性也是用`CSS`像素度量的。你想知道文档已经向上滚动了多少，不管缩放状态是什么。

理论上，如果用户向上滚动，然后放大，`windw.pageX / YOffset`会改变。然而，浏览器试图保持网页的一致性，通过当用户放大时，在可见页面的顶部保持相同的元素来实现。这并不总是完美的，但这意味着在实践中`window.pageX/YOffset`并没有真正改变：已经滚动出窗口的`CSS`像素数量保持(大致)相同。

![img](https://www.quirksmode.org/mobile/pix/viewport/desktop_page_zoomed.jpg)

### 概念: 视口the `viewport`

在我们继续讨论更多的JavaScript属性之前，我们必须引入另一个概念：视口 `the viewport`。

`viewport`的功能是约束<html>元素，它是网页最上面的包含块。

这听起来可能有点模糊，所以这里有一个实际的例子。假设你有一个流式布局，其中一个侧边栏宽度为10%。现在，边栏会随着你调整浏览器窗口的大小而整齐地增长和收缩。但这到底是怎么回事呢?

从技术上讲，侧边栏的宽度是其父栏的10%。我们假设这是<body>(并且你没有给它设置宽度)。问题是<body>的宽度是多少。

通常情况下，所有块级元素的宽度都是其父元素的100%(当然也有例外，但我们先忽略它们)。所以<body>和它的父元素<html>一样宽。

<html>元素有多宽？它和浏览器窗口一样宽。这就是为什么你的侧边栏`width:10%`将跨越整个浏览器窗口的10%。所有的web开发人员都直观地知道并使用这个事实。

你可能不知道这在理论上是如何运作的。理论上，<html>元素的宽度受`viewport`宽度的限制。<html>元素占`viewport`宽度的100%。

`viewport`，反过来，完全等于浏览器窗口：它是这样定义的。`viewport`不是HTML构造，因此不能通过`CSS`来影响它。它只是桌面浏览器窗口的宽度和高度。在移动平台上则要复杂得多。

### 结果

这种情况产生了一些奇怪的后果。你可以在这个网站上看到其中一个。一直滚动到顶部，放大两到三倍，这样网站的内容就会溢出浏览器窗口。

现在向右滚动，你会看到网站顶部的蓝色条不再正确排列。

![img](https://www.quirksmode.org/mobile/pix/viewport/desktop_htmlbehaviour.jpg)

这种行为是`viewport`定义方式的结果。我给顶部的蓝色条定义`width:100%`。是谁的100%？<html>元素，它与`viewport`一样宽，`viewport`与浏览器窗口一样宽。

关键点在于：虽然在100%缩放时可以正常工作，但现在我们已经缩小了`viewport`，它比我的网站的总宽度还小。这本身并不重要，内容现在从<html>元素溢出，但该元素具有`overflow: visible`，这意味着溢出的内容将在任何情况下显示。

但是蓝色的条并没有溢出来。因为我给它设置了`width:100%`，毕竟，浏览器通过把`viewport`的宽度应用在蓝色条上来遵守这一设置。他们并不关心宽度现在太窄了。

![img](https://www.quirksmode.org/mobile/pix/viewport/desktop_100percent.jpg)

### 文档宽度?

我真正需要知道的是整个页面的内容有多宽，包括那些突出的部分。据我所知，不可能找到这个值(除非你去计算页面上所有元素的宽度和页边距，但说得委婉些，这很容易出错)。

我相信我们需要一个JavaScript属性对，来给出我所说的“文档宽度”(显然是`CSS`像素)。

![img](https://www.quirksmode.org/mobile/pix/viewport/desktop_documentwidth.jpg)

如果我们真的觉得很难，为什么不把这个值也暴露给`CSS`呢？我希望能够使蓝色条的`width:100%`属性依赖于文档的宽度，而不是<html>元素的宽度。(不过，这肯定很棘手，如果无法实现，我也不会感到惊讶。)

浏览器厂商们，你们怎么看?

### 测量`viewport`

你可能想知道视口的尺寸。它们可以在`document.documentElement.clientWidth`和`-Height`中找到。

![img](https://www.quirksmode.org/mobile/pix/viewport/desktop_client.jpg)

如果了解DOM，就会了解`document.documentElement`实际上是<html>元素：任何`html`文档的根元素。然而，可以说视口更高一级；它是包含<html>元素的元素。如果给<html>元素一个宽度，这可能很重要。(顺便说一下，我不建议这么做，但这是可能的。)

在这种情况下，`document.documentElement.clientWidth`和`-Height`仍然给出视口的尺寸，而不是<html>元素的尺寸。(这是一个特殊的规则，只适用于这个元素，只适用于这个属性对。在所有其他情况下，使用元素的实际宽度。)

![img](https://www.quirksmode.org/mobile/pix/viewport/desktop_client_smallpage.jpg)

因此`document.documentElement.clientWidth`和`-Height`总是给出视口的尺寸，而不管<html>元素的尺寸。

### 两个属性对

但是视窗宽度的尺寸不也是由`window.innerWidth/Height`给出的吗?是也不是。

这两个属性对之间有一个形式上的区别：`document.documentElement.clientWidth`和`-Height`不包括滚动条，而`window.innerWidth/Height`包括。不过，这基本上是一种吹毛求疵。

我们有两个属性对的现状是浏览器之战遗留下来的。那时`Netscape`只支持`window.innerWidth/Height`而IE仅支持`document.documentElement.clientWidth`和`-Height`。从那时起，所有其他浏览器都开始支持`clientWidth/Height`，但IE没有选择`window.innerWidth/Height`。

在台式机上有两个可用的属性对是一个小麻烦——但在移动设备上却是一个福音，正如我们将看到的。

### 测量<html>元素

所以`clientWidth/Height`给出了所有情况下的视口尺寸。但是我们在哪里可以找到<html>元素本身的尺寸呢?它们存储在`document.documentelement.offsewidth`和`-Height`中。

![img](https://www.quirksmode.org/mobile/pix/viewport/desktop_offset.jpg)

这些属性真正地让你以块级元素的方式访问<html>元素；如果你设置`width`，`offsetWidth`将反映它。

![img](https://www.quirksmode.org/mobile/pix/viewport/desktop_offset_smallpage.jpg)

### 事件的坐标

然后是事件坐标。当鼠标事件发生时，至少会暴露5个属性对以提供有关事件确切位置的信息。对于我们的讨论，其中三个是重要的：

> 1. `pageX/Y`以`CSS`像素给出元素相对于<html>元素的坐标。
> 2. `clientX/Y`以`CSS`像素给出元素相对于`viewport`的元素坐标。
> 3. `screenX/Y` 以`CSS`像素给出相对于屏幕的坐标。

![img](https://www.quirksmode.org/mobile/pix/viewport/desktop_pageXY.jpg)

90%的时间你都会都使用`pageX/Y`；通常，你想知道事件相对于文档的位置。另外10%的时间你会使用`clientX/Y`。您永远不需要知道相对于屏幕的事件坐标。

### 媒体查询

最后，一些关于媒体查询的词汇。其思想非常简单：您可以定义特殊的`CSS`规则，这些规则只在页面宽度大于、等于或小于特定大小时执行。例如：

```css
div.sidebar {
	width: 300px;
}

@media all and (max-width: 400px) {
	// styles assigned when width is smaller than 400px;
	div.sidebar {
		width: 100px;
	}

}
```

现在侧边栏是`300px`宽，除非宽度小于`400px`，在这种情况下侧边栏变成`100px`宽。

问题当然是：我们测量的宽度是多少?

有两个相关的媒体查询:``width/height` 和 `device-width/device-height`。

1. `width/height` 使用与`documentElement.clientwidth /Height`（换句话说，`viewport`）。它适用于`CSS`像素。
2. `device-width/device-height` 使用与`screen.width/height`相同的值。(换句话说，屏幕)。它适用于设备像素。

![img](https://www.quirksmode.org/mobile/pix/viewport/desktop_mediaqueries.jpg)

你应该使用哪一种？显而易见，`width`。Web开发人员对设备宽度不感兴趣；重要的是浏览器窗口的宽度。

所以在桌面使用宽度，而忘记设备宽度。我们将看到，移动平台的情况更加混乱。

### 结论

这就是我们对桌面浏览器行为的探索。本系列的第二部分将这些概念移植到移动设备，并强调了与桌面设备的一些重要区别。
