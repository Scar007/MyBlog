---
title: 使用Vue脚手架(vue-cli)从零搭建一个vue项目(包含vue项目结构展示)
subtitle: vue
date: 2018-4-22
categories: VUE
tags: 
    - VUE
---
注：在搭建项目之前，请先安装一些全局的工具（如：node，vue-cli等）

### node安装：[node官网](https://nodejs.org/en/)
下载并安装node即可，安装node以后就可以正常使用npm命令

### 全局安装vue-cli工具：
```javascript
npm install vue-cli -g
```

## 开始创建项目：

#### 找一个合适的位置，打开命令窗口，使用vue初始化基于webpack的新项目
<img src="https://images2018.cnblogs.com/blog/1140938/201804/1140938-20180425190602586-550899917.png"/>
```javascript
vue init webpack vue-demo  //注意名称太长的话它会有错误提示，就像VueDemo 我们可以写成vue-demo就可以了
```
 如上图，它会又各种提示，还有需要安装的东西，可以自行按照自己的需求去安装东西。

 生成以后它会在你刚开始选择的位置生成一个现成的vue的一个项目架子，就像这样：
<img src="https://images2018.cnblogs.com/blog/1140938/201804/1140938-20180425191234198-1947303353.png"/>

## 启动项目：
```javascript
npm run dev
```
<img src="https://images2018.cnblogs.com/blog/1140938/201804/1140938-20180425191429550-1328316085.png"/>
根据上边的地址在浏览器菜单栏里边打开项目即可>
<img src="https://images2018.cnblogs.com/blog/1140938/201804/1140938-20180425191623568-171220875.png"/>

项目启动成功了，我们可以继续我们项目中所需要的工具，如：
```javascript
npm install vue-router --save （路由管理模块）
npm install vuex --save （状态管理模块）
npm install vue-resource --save （网路请求模块）
```

#### 以下是脚手架生成的项目结构：
```javascript
|-- build                            // 项目构建(webpack)相关代码
|   |-- build.js                     // 生产环境构建代码
|   |-- check-version.js             // 检查node、npm等版本
|   |-- dev-client.js                // 热重载相关
|   |-- dev-server.js                // 构建本地服务器
|   |-- utils.js                     // 构建工具相关
|   |-- webpack.base.conf.js         // webpack基础配置
|   |-- webpack.dev.conf.js          // webpack开发环境配置
|   |-- webpack.prod.conf.js         // webpack生产环境配置
|-- config                           // 项目开发环境配置
|   |-- dev.env.js                   // 开发环境变量
|   |-- index.js                     // 项目一些配置变量
|   |-- prod.env.js                  // 生产环境变量
|   |-- test.env.js                  // 测试环境变量
|-- src                              // 源码目录（里面可以自定义文件结构）
|   |-- components                     // vue公共组件
|   |-- store                          // vuex的状态管理
|   |-- App.vue                        // 页面入口文件
|   |-- main.js                        // 程序入口文件，加载各种公共组件
|-- static                           // 存放静态文件的文件夹，比如一些图片，json数据等
|-- .babelrc                         // ES6语法编译配置
|-- .editorconfig                    // 定义代码格式
|-- .gitignore                       // git上传需要忽略的文件格式
|-- README.md                        // 项目说明
|-- index.html                       // 入口页面
|-- package.json                     // 项目基本信息，第三方依赖的文件列表
```

