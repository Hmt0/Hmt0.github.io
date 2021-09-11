---
title: webpack入门六、性能调优
date: 2021-09-10 22:40:22
tags: webpack
---

# `webpack`性能调优

#### 学习目标

- 打包结果优化
- 构建过程优化
- tree-shaking

`webpack`在生产环境下会自动压缩代码

#### 预制`terser plugin`：实现插件定制

```javascript
const TerserPlugin = require('terser-webpack-plugin')
```

配置：

```javascript
 optimization: {
        minimizer: [new TerserPlugin({
            // 使用缓存，加快构建速度
            cache: true,
            parallel: true,
            // 打包多线程
            TerserOptions: {
                compress: {
                    // 移除无用代码,debugger、console等
                    unused: true,
                    drop_debugger: true,
                    drop_console: true,
                    dead_code: true
                }
            }
        })]
    },
```

打包结果可视化分析：`webpack.bundle-analyzer`，可以看到哪一部分占的体积更大

```
npm install webpack-bundle-analyzer
```

配置：

```javascript
new BundleAnalyzerPlugin()
```

报错

> [webpack-cli] TypeError: BundleAnalyzerPlugin is not a constructor

解决：解构赋值引用

#### 其他调优方式：

- 减少解析：配置不解析的文件：


```
noParse: /node_modules\/(jquery\.js)/,
```

- 减少查找：include只打包某些文件


- 利用多线程提高构建速度，并发执行子进程：

1. `thread-loader`把loader放在worker线程池里，必须放在所有`loader`之前
2. `HappyPack`：多进程

```javascript
// 根据cpu数量创建线程池
const happyThreadPool = HappyPack.ThreadPool({size: os.cpus().length})
// url-loader file-loader不支持happypack
```

- 预编译：

- 缓存：fast-sass-loader并行处理sass文件


- source-map


#### tree-shaking

消除无用JavaScript代码`DCE`， 例如：引用未执行的模块

development：去掉无用引用

run build：直接去掉

总结：`webpack`是什么

- 前端发展的产物

- 模块化打包方案

- 工程化方案 ，例如`create-react-app`也是基于`webpack`

补充：了解进程与线程的概念，`webpack`优化是体现工作能力的地方
