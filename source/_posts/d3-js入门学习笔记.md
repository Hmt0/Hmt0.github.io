---
title: d3.js入门学习笔记
date: 2022-01-21 14:30:55
tags: d3
---

> `D3js` 是一个可以基于数据来操作文档的 `JavaScript` 库。可以帮助你使用 `HTML`, `CSS`, `SVG` 以及 `Canvas` 来展示数据。`D3` 遵循现有的 `Web` 标准，可以不需要其他任何框架独立运行在现代浏览器中，它结合强大的可视化组件来驱动 `DOM` 操作。

### selections选择集

- `select`：选择`html`节点

### 动态属性

- ```javascript
  d3.selectAll("div")
  
  .data(dataset)
  
  .enter()
  
  .append("div")
  ```

填充数组元素，补充缺少的对应节点

- `attr`：添加类名或者属性

### 直方图

#### 练习1：**为集合中的每个数据点创建一个数据条**

```javascript
<body>
  <script>
    const dataset = [12, 31, 22, 17, 25, 18, 29, 14, 9];

    const w = 500;
    const h = 100;

    const svg = d3.select("body")
                  .append("svg")
                  .attr("width", w)
                  .attr("height", h);

    svg.selectAll("rect")
       // 在这行下面添加代码
      .data(dataset)
      .enter()
      .append("rect")
       // 在这行上面添加代码
       .attr("x", 0)
       .attr("y", 0)
       .attr("width", 25)
       .attr("height", 100);
  </script>
</body>
```

在 `SVG` 中，坐标轴的原点在左上角。 `x` 坐标为 0 将图形放在` SVG` 区域的左边缘， `y` 坐标为 0 将图形放在 `SVG` 区域的上边缘。 `x` 值增大矩形将向右移动， `y` 值增大矩形将向下移动。

#### 练习2：**反转 SVG 元素**

```
<body>
  <script>
    const dataset = [12, 31, 22, 17, 25, 18, 29, 14, 9];

    const w = 500;
    const h = 100;

    const svg = d3.select("body")
                  .append("svg")
                  .attr("width", w)
                  .attr("height", h);

    svg.selectAll("rect")
       .data(dataset)
       .enter()
       .append("rect")
       .attr("x", (d, i) => i * 30)
       .attr("y", (d, i) => h - d * 3)
       .attr("width", 25)
       .attr("height", (d, i) => 3 * d);
  </script>
</body>
```

#### 练习3. **给 `D3` 元素添加悬停效果**

```html
<style>
  .bar:hover {
    fill: brown;
  }
</style>
<body>
  <script>
    const dataset = [12, 31, 22, 17, 25, 18, 29, 14, 9];

    const w = 500;
    const h = 100;

    const svg = d3.select("body")
                  .append("svg")
                  .attr("width", w)
                  .attr("height", h);

    svg.selectAll("rect")
       .data(dataset)
       .enter()
       .append("rect")
       .attr("x", (d, i) => i * 30)
       .attr("y", (d, i) => h - 3 * d)
       .attr("width", 25)
       .attr("height", (d, i) => 3 * d)
       .attr("fill", "navy")
       // 在这行下面添加代码
        .attr("class", "bar")
       // 在这行上面添加代码

    svg.selectAll("text")
       .data(dataset)
       .enter()
       .append("text")
       .text((d) => d)
       .attr("x", (d, i) => i * 30)
       .attr("y", (d, i) => h - (3 * d) - 3);

  </script>
</body>
```

#### 练习4.**给 `D3 `元素添加工具提示**

```html
<style>
  .bar:hover {
    fill: brown;
  }
</style>
<body>
  <script>
    const dataset = [12, 31, 22, 17, 25, 18, 29, 14, 9];

    const w = 500;
    const h = 100;

    const svg = d3.select("body")
                  .append("svg")
                  .attr("width", w)
                  .attr("height", h);

    svg.selectAll("rect")
       .data(dataset)
       .enter()
       .append("rect")
       .attr("x", (d, i) => i * 30)
       .attr("y", (d, i) => h - 3 * d)
       .attr("width", 25)
       .attr("height", (d, i) => d * 3)
       .attr("fill", "navy")
       .attr("class", "bar")
       // 在这行下面添加代码
        .append("title")
        .text(d => d)


       // 在这行上面添加代码

    svg.selectAll("text")
       .data(dataset)
       .enter()
       .append("text")
       .text((d) => d)
       .attr("x", (d, i) => i * 30)
       .attr("y", (d, i) => h - (d * 3 + 3))

  </script>
</body>
```

### 散点图

#### 练习1.**给 Circle 元素添加属性**

```html
<body>
  <script>
    const dataset = [
                  [ 34,    78 ],
                  [ 109,   280 ],
                  [ 310,   120 ],
                  [ 79,    411 ],
                  [ 420,   220 ],
                  [ 233,   145 ],
                  [ 333,   96 ],
                  [ 222,   333 ],
                  [ 78,    320 ],
                  [ 21,    123 ]
                ];


    const w = 500;
    const h = 500;

    const svg = d3.select("body")
                  .append("svg")
                  .attr("width", w)
                  .attr("height", h);

    svg.selectAll("circle")
       .data(dataset)
       .enter()
       .append("circle")
       // 在这行下面添加代码
      .attr("cx", d => d[0])
      .attr("cy", d => w - d[1])
      .attr("r", 5)

       // 在这行上面添加代码

  </script>
</body>
```

#### **按比例设置域和范围**

缩放的输入信息：*域*`domain`

输出信息：*范围*`range`

查找数据中最大值：`max`

#### 练习2：**使用动态比例**

**注意：**记得保持绘图在右上角。 当你为 y 坐标设置 range 时，大的值（height 减去 padding）是第一个参数，小的值是第二个参数。

```javascript
<body>
  <script>
    const dataset = [
                  [ 34,    78 ],
                  [ 109,   280 ],
                  [ 310,   120 ],
                  [ 79,    411 ],
                  [ 420,   220 ],
                  [ 233,   145 ],
                  [ 333,   96 ],
                  [ 222,   333 ],
                  [ 78,    320 ],
                  [ 21,    123 ]
                ];

    const w = 500;
    const h = 500;

    // 画布边线和图表之间的 padding
    const padding = 30;

    // 创建 x 和 y 轴

    const xScale = d3.scaleLinear()
                    .domain([0, d3.max(dataset, (d) => d[0])])
                    .range([padding, w - padding]);

    // 在这行下面添加代码

    const yScale = d3.scaleLinear().domain([0, d3.max(dataset, d => d[1])])
    .range([h - padding, padding]);

    // 在这行上面添加代码

    const output = yScale(411); // 返回 30
    d3.select("body")
      .append("h2")
      .text(output)
  </script>
</body>
```

#### 练习3：**使用预定义的比例放置元素**

```javascript
<body>
  <script>
    const dataset = [
                  [ 34,     78 ],
                  [ 109,   280 ],
                  [ 310,   120 ],
                  [ 79,   411 ],
                  [ 420,   220 ],
                  [ 233,   145 ],
                  [ 333,   96 ],
                  [ 222,    333 ],
                  [ 78,    320 ],
                  [ 21,   123 ]
                ];

    const w = 500;
    const h = 500;
    const padding = 60;

    const xScale = d3.scaleLinear()
                     .domain([0, d3.max(dataset, (d) => d[0])])
                     .range([padding, w - padding]);

    const yScale = d3.scaleLinear()
                     .domain([0, d3.max(dataset, (d) => d[1])])
                     .range([h - padding, padding]);

    const svg = d3.select("body")
                  .append("svg")
                  .attr("width", w)
                  .attr("height", h);

    svg.selectAll("circle")
       .data(dataset)
       .enter()
       .append("circle")
       // 在这行下面添加代码
        .attr("cx", d => xScale(d[0]))
        .attr("cy", d => yScale(d[1]))
        .attr("r", 5)

       // 在这行上面添加代码

    svg.selectAll("text")
       .data(dataset)
       .enter()
       .append("text")
       .text((d) =>  (d[0] + ", "
 + d[1]))
       // 在这行下面添加代码
  .attr("x", d => xScale(d[0] + 10))
  .attr("y", d => yScale(d[1]))


       // 在这行上面添加代码
  </script>
</body>
```

#### 练习4：**添加坐标轴到视图中**

```javascript
<body>
  <script>
    const dataset = [
                  [ 34,     78 ],
                  [ 109,   280 ],
                  [ 310,   120 ],
                  [ 79,   411 ],
                  [ 420,   220 ],
                  [ 233,   145 ],
                  [ 333,   96 ],
                  [ 222,    333 ],
                  [ 78,    320 ],
                  [ 21,   123 ]
                ];

    const w = 500;
    const h = 500;
    const padding = 60;

    const xScale = d3.scaleLinear()
                     .domain([0, d3.max(dataset, (d) => d[0])])
                     .range([padding, w - padding]);

    const yScale = d3.scaleLinear()
                     .domain([0, d3.max(dataset, (d) => d[1])])
                     .range([h - padding, padding]);

    const svg = d3.select("body")
                  .append("svg")
                  .attr("width", w)
                  .attr("height", h);

    svg.selectAll("circle")
       .data(dataset)
       .enter()
       .append("circle")
       .attr("cx", (d) => xScale(d[0]))
       .attr("cy",(d) => yScale(d[1]))
       .attr("r", (d) => 5);

    svg.selectAll("text")
       .data(dataset)
       .enter()
       .append("text")
       .text((d) =>  (d[0] + "," + d[1]))
       .attr("x", (d) => xScale(d[0] + 10))
       .attr("y", (d) => yScale(d[1]))

    const xAxis = d3.axisBottom(xScale);
    // 在这行下面添加代码
    const yAxis = d3.axisLeft(yScale);
    // 在这行上面添加代码

    svg.append("g")
       .attr("transform", "translate(0," + (h - padding) + ")")
       .call(xAxis);

    // 在这行下面添加代码
   svg.append("g").attr("transform", "translate(" + padding + ", 0)")
   .call(yAxis)


    // 在这行上面添加代码

  </script>
</body>
```

