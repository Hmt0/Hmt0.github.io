---
title: for...of 与 for...in的区别
date: 2021-09-10 16:44:47

tags: JavaScript
---

`for...of`和`for...in`的作用都是迭代。它们之间的主要区别是迭代的内容。

`for...in`语句以任意顺序迭代**对象**的可枚举**属性**，包括实例属性和原型属性。

`for...of`语句迭代**可迭代对象**定义的要迭代的**值**。

那么，可迭代对象有哪些呢？包括：`String`，`Array`，`array-like objects`，`TypedArray`，`Map`，`Set`。

注意Object实例不是可迭代对象，所以不能用`for...of`迭代它的值，但可以用`for...in`枚举它的属性。

下面的例子展示了`for...of`循环和`for...in`循环应用在数组上的区别：

```javascript
Object.prototype.objCustom = function() {};
Array.prototype.arrCustom = function() {};
const iterable = [3, 5, 7];
iterable.foo = 'hello';

for (const i in iterable) {
    console.log(i); // logs "0", "1", "2", "foo"， "arrCustom", "objCustom"
}

for (const i in iterable) {
    if(iterable.hasOwnProperty(i)) {
    console.log(i); // logs "0", "1", "2", "foo"
    }
}

for (const i of iterable) {
    console.log(i); // logs 3, 5, 7
}
```

让我们一步一步地看一下上面的代码。

```javascript
Object.prototype.objCustom = function() {};
Array.prototype.arrCustom = function() {};
const iterable = [3, 5, 7];
iterable.foo = 'hello';
```

每个对象实例都会继承`objectCustom`属性，并且每个Array对象的实例都会继承`arrCustom`属性，这是因为这些属性已分别添加到`Onject.prototype`和`Array.prototype`。由于继承和原型链，可迭代对象继承了属性`objCustom`和`arrCustom`。

```javascript
for (const i in iterable) {
  console.log(i); // logs 0, 1, 2, "foo", "arrCustom", "objCustom"
}
```

这个循环只以任意顺序打印了可迭代对象的枚举属性。它不会打印数组**元素**`3`，`5`，`7`或者`hello`，因为它们不是可枚举属性，**实际上它们根本不是属性**，它们是值。它所打印的数组的下标，还有`arrCustom`和`objCustom`才是属性。

```javascript
for (const i in iterable) {
  if (iterable.hasOwnProperty(i)) {
    console.log(i); // logs 0, 1, 2, "foo"
  }
}
```

这个循环和上一个相似，但它使用`hasOwnProperty()`来检查可枚举属性是否是对象自身的，不是继承的。如果是，则打印该属性。属性`0`，`1`，`2`和`foo`被输出，因为他们是自有属性(不是继承的)。属性`arrCustom`和`objCustom`没有被输出，因为他们是继承的。

```javascript
for (const i of iterable) {
  console.log(i); // logs 3, 5, 7
}
```

这个循环迭代并打印了可迭代的值，作为可迭代对象所定义的要迭代的值。对象的元素 3、5、7被打印，但没有打印对象的任何属性。

补充：JavaScript规范定义的数据属性[[Enumerable]]表示属性是否可以通过`for...in`循环返回。默认情况下，所有直接定义在对象上的属性的这个特性都是true。
