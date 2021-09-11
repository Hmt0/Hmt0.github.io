---
title: webpack入门三、配置开发环境-npm与包管理器
date: 2021-09-10 20:58:27
tags: webpack
---



# 配置开发环境——`npm`与包管理器

#### 学习目标

- 理解包管理器
- 熟悉`npm`核心特性
- 理解`npm`”仓库“与”依赖“的概念
- 理解`npm`”语义化版本“
- 掌握使用`npm`自定义工程脚本的方法

#### 什么是包管理器？

使开发者能够便捷的管理代码和分发代码的工具

###### `npm`依赖`node`环境，安装`node`会自动安装`npm`，查看`node`版本和`npm`版本：

```shell
node -v // 查看node版本
npm -v // 查看npm版本
```

######  初始化工程： 

```
npm-init 
```

###### `package.json`：

```json
{
  "name": "learn-webpack", // 包名称
  "version": "1.0.0", // 版本号
  "description": "",
  "main": "index.js", // 执行入口
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
  }, // 自定义脚本，自动化执行命令
  "author": "",
  "license": "ISC"
}
```

######  设置国内镜像：

```powershell
npm config set registry https://registry.npm.taobao.org
```

###### 安装包：

```
npm install lodash --save
```

`--save`或` -s`将下载的包写入`package.json`的*生产环境*下的依赖`dependencies`中，`npm`5.0版本之后默认`--save`

```powershell
npm install jquary --save-dev
-d
```

`--save-dev`或`-d`添加到*开发环境*下的依赖`devDepencies`，主要是构建工具，质量检测工具比如`eslint`等本地开发用到的包。

项目迁移时，执行`npm install`即可安装`开发环境和生产环境下的包。

```powershell
npm install --only=dev
```

只安装开发环境下的包

```
npm install --only=prod
```

只安装生产环境中的包，由此可以实现环境区分。

###### 删除并重装

```powershell
rm -rf node_modules && npm install
```

#### 版本号，语义化版本

- ^version: 中版本和小版本

  ^1.0.1 -> 1.x.x

- ~version: 小版本

  ~1.0.1 -> 1.0.x

  方便推送小的新版本（fix bug等)

- version：固定特定版本

#### scripts自定义命令执行脚本

```javascript
{    
    test: "echo \"Error: no test specified\" && exit 1"
    dev: "webpack-dev-sever"//启动ewbpack服务	
    build: 'eslint ./src && webpack'
}
```

#### `npm install`的过程：

https://mp.weixin.qq.com/s/5tmND0G_ZkYVR7Dmug0ugQ

- 寻找包版本信息文件`package.json`，依照它来进行安装
- 查`package.json`中的依赖，并检查项目中其他版本信息文件，如果不存在，则按照`package.json`的依赖进行安装，并生成版本信息文件
- 如果发现了新包，就更新版本信息文件

