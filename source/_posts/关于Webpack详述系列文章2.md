---
title: 关于Webpack详述系列文章 （第二篇）
subtitle: wp
date: 2018-5-23
categories: WebPack
tags: 
    - WebPack
---
<b> [WebPack官网](https://webpack-china.org/)</b>

## 1.缩小文件搜索范围

#### 1.1.1 include & exclude
```javascript
module:{
        rules:[
            {
                 test:/\.js$/,
                 use:['babel-loader?cacheDirectory'],
                include:path.resolve(__dirname,'src'),
                exclude:/node_modules/
            }
        ]
    }
```

#### 1.1.2 resolve.modules
```javascript
resolve: {
        modules: [path.resolve(__dirname, 'node_modules')]
},
```

#### 1.1.3 resolve.mainFields
+ mainFields用于配置第三方模块使用那个入口文件 isomorphic-fetch
    + 当target为web或webworker时，值是["browswer","module","main"]
    + 当target为其他情况时，值是["module","main"] ```js resolve: {
    + mainFields:['main'] }, `　　　

#### 1.1.4 resolve.alias
+ resolve.alias配置项通过别名来把原导入路径映射成一个新的导入路径 此优化方法会影响使用Tree-Shaking去除无效代码
```javascript
alias: {
            'react': path.resolve(__dirname, './node_modules/react/cjs/eact.production.min.js')
        }
```

#### 1.1.5 resolve.extensions 
+ 在导入语句没带文件后缀时，Webpack会自动带上后缀后去尝试询问文件是否存在 默认后缀是 extensions: ['.js', '.json']
    + 后缀列表尽可能小
    + 频率最高的往前方
    + 导出语句里尽可能带上后缀
    ```javascript
    resolve: {
        extensions: ['js']
    },
    ```

#### 1.1.6 module.noParse
+ module.noParse 配置项可以让 Webpack 忽略对部分没采用模块化的文件的递归解析处理
```javascript
module: {
        noParse: [/react\.min\.js/]
    }
```
> 被忽略掉的文件里不应该包含 import 、 require 、 define 等模块化语句

## 2.DLL 
+ .dll 为后缀的文件称为动态链接库，在一个动态链接库中可以包含给其他模块调用的函数和数据
    + 把基础模块独立出来打包到单独的动态连接库里
    + 当需要导入的模块在动态连接库里的时候，模块不能再次被打包，而是去动态连接库里获取 dll-plugin

#### 2.1 定义Dll
+ DllPlugin插件： 用于打包出一个个动态连接库
+ DllReferencePlugin: 在配置文件中引入DllPlugin插件打包好的动态连接库
```javascript
module.exports = {
    entry: {
        react: ['react'] //react模块打包到一个动态连接库
    },
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].dll.js', //输出动态连接库的文件名称
        library: '_dll_[name]' //全局变量名称
    },
    plugins: [
        new webpack.DllPlugin({
            name: '_dll_[name]', //和output.library中一致，值就是输出的manifest.json中的 name值
            path: path.join(__dirname, 'dist', '[name].manifest.json')
        })
    ]
}
```
```javascript
webpack --config webpack.dll.config.js --mode production
```

#### 2.2 使用动态链接库文件
```javascript
plugins: [
        new webpack.DllReferencePlugin({
            manifest: require(path.join(__dirname, 'dist', 'react.manifest.json')),
        })
    ],
```
```javascript
webpack --config webpack.config.js --mode development
```

## 3. HappyPack
+ HappyPack就能让Webpack把任务分解给多个子进程去并发的执行，子进程处理完后再把结果发送给主进程。 happypack
```javascript
npm i happypack@next -D
```
```javasript
module: {
        rules: [{
            test: /\.js$/,
            //把对.js文件的处理转交给id为babel的HappyPack实例
          use: 'happypack/loader?id=babel',
            include: path.resolve(__dirname, 'src'),
            exclude: /node_modules/
        }, {
            //把对.css文件的处理转交给id为css的HappyPack实例
            test: /\.css$/,
           use: 'happypack/loader?id=css',
            include: path.resolve(__dirname, 'src')
        }],
        noParse: [/react\.min\.js/]
    },
```
```javascript
plugins: [
        //用唯一的标识符id来代表当前的HappyPack是用来处理一类特定文件
        new HappyPack({
            id: 'babel',
            //如何处理.js文件，和rules里的配置相同
            loaders: [{
                loader: 'babel-loader',
                query: {
                    presets: [
                        "env", "react"
                    ]
                }
            }]
        }),
        new HappyPack({
            id: 'css',
            loaders: ['style-loader', 'css-loader'],
            threads: 4, //代表开启几个子进程去处理这一类型的文件
            verbose: true //是否允许输出日子
        })
    ],
```

## 4. ParallelUglifyPlugin
+ ParallelUglifyPlugin可以把对JS文件的串行压缩变为开启多个子进程并行执行
```javascript
npm i -D webpack-parallel-uglify-plugin
```
```javascript
new ParallelUglifyPlugin({
            workerCount: 3, //开启几个子进程去并发的执行压缩。默认是当前运行电脑的 CPU 核数减去1
            uglifyJS: {
                output: {
                    beautify: false, //不需要格式化
                    comments: false, //不保留注释
                },
                compress: {
                    warnings: false, // 在UglifyJs删除没有用到的代码时不输出警告
                    drop_console: true, // 删除所有的 `console` 语句，可以兼容ie浏览器
                    collapse_vars: true, // 内嵌定义了但是只用到一次的变量
                    reduce_vars: true, // 提取出出现多次但是没有定义成变量去引用的静态值
                }
            },
        })
```

## 5. 服务器自动刷新
+ 我们可以监听到本地源码文件发生变化时，自动重新构建出可运行的代码后再刷新浏览器

#### 5.1 文件监听
```javascript
watch: true, //只有在开启监听模式时，watchOptions才有意义
 watchOptions: {
    ignored: /node_modules/,
    aggregateTimeout: 300, //监听到变化发生后等300ms再去执行动作，防止文件更新太快导致编译频率太高
    poll: 1000 //通过不停的询问文件是否改变来判断文件是否发生变化，默认每秒询问1000次
 }
```

#### 5.2 文件监听流程
+ webpack定时获取文件的更新时间，并跟上次保存的时间进行比对，不一致就表示发生了变化,poll就用来配置每秒问多少次
+ 当检测文件不再发生变化，会先缓存起来，等待一段时间后之后再通知监听者，这个等待时间通过aggregateTimeout配置
+ webpack只会监听entry依赖的文件
+ 我们需要尽可能减少需要监听的文件数量和检查频率，当然频率的降低会导致灵敏度下降

#### 5.3 自动刷新浏览器
```javascript
devServer: {
        contentBase: './dist',
        inline: true
    },
```
webpack负责监听文件变化，webpack-dev-server负责刷新浏览器 这些文件会被打包到chunk中，它们会代理客户端向服务器发起WebSocket连接
```javascript
(webpack)-dev-server/client/overlay.js 3.58 KiB {0} [built]
 (webpack)-dev-server/client/socket.js 1.05 KiB {0} [built]
./node_modules/loglevel/lib/loglevel.js 7.68 KiB {0} [built]
./node_modules/strip-ansi/index.js 161 bytes {0} [built]
./node_modules/url/url.js 22.8 KiB {0} [built]
(webpack)-dev-server/client?http://localhost:8080 7.75 KiB {0} [built]
 multi (webpack)-dev-server/client?http://localhost:8080 ./src/index.js 40 bytes {0} [built]
```

#### 5.4 模块热替换 
+ 模块热替换(Hot Module Replacement)的技术可在不刷新整个网页的情况下只更新指定的模块 原理是当一个源码发生变化时，只重新编译发生变化的模块，再用新输出的模块替换掉浏览器中对应的老模块
    + 反应更快，时间更短
    + 不刷新网页可以保留网页运行状态
    ```javascript
    devServer: {
       hot:true
    }
    ```
    ```javascript
    [./node_modules/webpack/hot sync ^\.\/log$] (webpack)/hot sync nonrecursive ^\.\/log$ 170 bytes {main} [built]
    [0] multi (webpack)-dev-server/client?http://localhost:8080 webpack/hot/dev-server ./src/index.js 52 bytes {main} [built]
    [./node_modules/webpack/hot/dev-server.js] (webpack)/hot/dev-server.js 1.66 KiB {main} [built]
    [./node_modules/webpack/hot/emitter.js] (webpack)/hot/emitter.js 77 bytes {main} [built]
    ```
    ```javascript
    if (module.hot) {
        module.hot.accept('./index.js', function () {
            console.log('accept index.js');
        });
    }
    ```
    优化模块热替换浏览器日志
    ```javascript
    plugins: [
        new webpack.NamedModulesPlugin(),
        new webpack.HotModuleReplacementPlugin(),
    ]
    ```
    + 监听更少的文件
    + 忽略掉 node_modules 目录下的文件

## 6.区分环境
+ 在开发网页的时候，一般都会有多套运行环境，例如：
    + 在开发过程中方便开发调试的环境。
    + 发布到线上给用户使用的运行环境。

#### 6.1 环境区别
+ 线上的代码被压缩
+ 开发环境可能会打印只有开发者才能看到的日志
+ 开发环境和线上环境后端数据接口可能不同

#### 6.2 如何使用 
```javascript
if(process.env.NODE_ENV == 'production'){
     console.log('生产环境');
}else{
    console.log('开发环境');
}
```
当你使用process模块的时候，webpack会把process模块打包进来
```javascript
new webpack.DefinePlugin({
             'process.env': {
                 NODE_ENV:JSON.stringify('production')
             }
         }),
```
定义环境变量的值时用 JSON.stringify 包裹字符串的原因是环境变量的值需要是一个由双引号包裹的字符串，而 JSON.stringify('production')的值正好等于'"production"'
```javascript
new webpack.DefinePlugin({
  'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
})
```

#### 6.3 三种区分环境文案
+ 通过npm命令区分
+ 通过环境变量区分 webpack-merge
+ 代码中区分

## 7. CDN 
+ CDN 又叫内容分发网络，通过把资源部署到世界各地，用户在访问时按照就近原则从离用户最近的服务器获取资源，从而加速资源的获取速度。
    + HTML文件不缓存，放在自己的服务器上，关闭自己服务器的缓存，静态资源的URL变成指向CDN服务器的地址
    + 静态的JavaScript、CSS、图片等文件开启CDN和缓存，并且文件名带上HASH值
    + 为了并行加载不阻塞，把不同的静态资源分配到不同的CDN服务器上
    ```javascript
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name]_[hash:8].js',
        publicPath: 'http://img.zhufengpeixun.cn'
    },
    ```

## 8.Tree Shaking
+ Tree Shaking 可以用来剔除JavaScript中用不上的死代码。它依赖静态的ES6模块化语法，例如通过import和export导入导出。
+ 使用Tree
    + 不要编译ES6模块
    ```javascipt
    {
                 loader: 'babel-loader',
                 query: {
                     presets: [
                         [
                              "env", {
                                  modules: false //含义是关闭 Babel 的模块转换功能，保留原本的 ES6 模块化语法
                              }
                         ],
                         "react"
                     ]
                 }
             }
    ```
    ```javascript
    webpack --display-used-exports
    ```
    ```javascript
    const UglifyJSPlugin = require('uglifyjs-webpack-plugin');

    plugins: [
    new UglifyJSPlugin()
    ]
    ```
    ```javascript
    webpack --display-used-exports --optimize-minimize
    webpack --mode  production
    ```

## 9.提取公共代码

#### 9.1 为什么需要提取公共代码
+ 大网站有多个页面，每个页面由于采用相同技术栈和样式代码，会包含很多公共代码，如果都包含进来会有问题
    + 相同的资源被重复的加载，浪费用户的流量和服务器的成本；
    + 每个页面需要加载的资源太大，导致网页首屏加载缓慢，影响用户体验。 如果能把公共代码抽离成单独文件进行加载能进行优化，可以减少网络传输流量，降低服务器成本

#### 9.2 如何提取
+ 基础类库，方便长期缓存
+ 页面之间的公用代码
+ 各个页面单独生成文件

如何使用 common-chunk-and-vendor-chunk
```javascript
entry: {
        pageA: './src/pageA',
        pageB: './src/pageB'
},

optimization: {
        splitChunks: {
            cacheGroups: {
                commons: {
                    chunks: "initial",
                    minChunks: 2,
                    maxInitialRequests: 5, // The default limit is too small to showcase the effect
                    minSize: 0 // This is example is too small to create commons chunks
                },
                vendor: {
                    test: /node_modules/,
                    chunks: "initial",
                    name: "vendor",
                    priority: 10,
                    enforce: true
                }
            }
        }
    },
```

## 10.开启 Scope Hoisting
+ Scope Hoisting 可以让 Webpack 打包出来的代码文件更小、运行的更快， 它又译作 "作用域提升"，是在 Webpack3 中新推出的功能。
    + 代码体积更小，因为函数申明语句会产生大量代码
    + 代码在运行时因为创建的函数作用域更少了，内存开销也随之变小 hello.js
```javascript
export default 'Hello';
```
```javascript
import str from './hello.js';
console.log(str);
```
```javascript
var util = ('Hello');
console.log(util);
```
函数由两个变成了一个，hello.js 中定义的内容被直接注入到了 main.js 中
```javascript
const ModuleConcatenationPlugin = require('webpack/lib/optimize/ModuleConcatenationPlugin');

module.exports = {
  resolve: {
    // 针对 Npm 中的第三方模块优先采用 jsnext:main 中指向的 ES6 模块化语法的文件
    mainFields: ['jsnext:main', 'browser', 'main']
  },
  plugins: [
    // 开启 Scope Hoisting
    new ModuleConcatenationPlugin(),
  ],
};
```
```javascript
--display-optimization-bailout
```

## 11.代码分离
+ 代码分离是 webpack 中最引人注目的特性之一。此特性能够把代码分离到不同的 bundle 中，然后可以按需加载或并行加载这些文件。 有三种常用的代码分离方法：
    + 入口起点：使用 entry 配置手动地分离代码。
    + 防止重复：使用 splitChunks 去重和分离 chunk。
    + 动态导入：通过模块的内联函数调用来分离代码。

#### 11.1 多个入口
```javascript
entry: {
  index: './src/index.js',
  another: './src/another-module.js'
}
```

#### 11.2 防止重复 
splitChunks可以将公共的依赖模块提提取到一个新生成的 chunk. common-chunk-and-vendor-chunk
```javascript
optimization: {
        splitChunks: {
            cacheGroups: {
                commons: {
                    chunks: "initial",
                    minChunks: 2
                },
                vendor: {
                    test: /node_modules/,
                    chunks: "initial",
                    name: "vendor",
                }
            }
```

####  11.3 动态导入和懒加载(dynamic imports)
+ 用户当前需要用什么功能就只加载这个功能对应的代码，也就是所谓的按需加载 在给单页应用做按需加载优化时，一般采用以下原则：
    + 对网站功能进行划分，每一类一个chunk
    + 对于首次打开页面需要的功能直接加载，尽快展示给用户
    + 某些依赖大量代码的功能点可以按需加载
    + 被分割出去的代码需要一个按需加载的时机
```javascript
document
    .getElementById('clickMe')
    .addEventListener('click', () => {
        import (/*webpackChunkName:"alert"*/
        './alert').then(alert => {
            console.log(alert);

            alert.default('hello');
        });
    });
```
```javascript
loaders: [
                {
                    loader: 'babel-loader',
                    query: {
                        presets: ["env", "stage-0", "react"]
                    }
                }
            ]    
```

## 12. webpack-dev-middleware 插件
webpack-dev-middleware 插件对更改的文件进行监控,编译,一般和 webpack-hot-middleware 配合使用，实现热加载功能 webpack-dev-middlewarewebpack-hot-middleware
```javascript
const path = require("path")
const express = require("express")
const webpack = require("webpack")
const webpackDevMiddleware = require("webpack-dev-middleware")
const webpackConfig = require('./webpack.config.js')
const app = express(),
            DIST_DIR = path.join(__dirname, "dist"),// 设置静态访问文件路径
            PORT = 9090, // 设置启动端口
            complier = webpack(webpackConfig)

app.use(webpackDevMiddleware(complier, {
//绑定中间件的公共路径,与webpack配置的路径相同
    publicPath: webpackConfig.output.publicPath,
    quiet: true  //向控制台显示内容
}))

// 这个方法和下边注释的方法作用一样，就是设置访问静态文件的路径
app.use(express.static(DIST_DIR))
app.listen(PORT,function(){
    console.log("成功启动：localhost:"+ PORT)
})
```

##  13. 输出分析 
+ profile：记录下构建过程中的耗时信息；
+ json：以 JSON 的格式输出构建结果，最后只输出一个 .json 文件，这个文件中包括所有构建相关的信息。
```javascript
webpack --profile --json > stats.json
```
+ Webpack 官方提供了一个可视化分析工具 Webpack Analyse
+ Modules：展示所有的模块，每个模块对应一个文件。并且还包含所有模块之间的依赖关系图、模块路径、模块ID、模块所属 + + Chunk、模块大小；
+ Chunks：展示所有的代码块，一个代码块中包含多个模块。并且还包含代码块的ID、名称、大小、每个代码块包含的模块数量，以及代码块之间的依赖关系图；
+ Assets：展示所有输出的文件资源，包括 .js、.css、图片等。并且还包括文件名称、大小、该文件来自哪个代码块；
+ Warnings：展示构建过程中出现的所有警告信息；
+ Errors：展示构建过程中出现的所有错误信息；
+ Hints：展示处理每个模块的过程中的耗时。

## 备注：
<b>libraryTarget 和 library：</b>
+ 当用 Webpack 去构建一个可以被其他模块导入使用的库时需要用到它们。　　　
    + output.libraryTarget 配置以何种方式导出库。
    + output.library 配置导出库的名称。 它们通常搭配在一起使用。

output.libraryTarget 是字符串的枚举类型，支持以下配置。　
<b>var (默认)</b>
　　编写的库将通过 var 被赋值给通过 library 指定名称的变量。
　　假如配置了 output.library='LibraryName'，则输出和使用的代码如下：
```javascript
// Webpack 输出的代码
var LibraryName = lib_code;

// 使用库的方法
LibraryName.doSomething();
假如 output.library 为空，则将直接输出：
```
lib_code 其中 lib_code 代指导出库的代码内容，是有返回值的一个自执行函数。

<b>commonjs</b>
编写的库将通过 CommonJS 规范导出。

假如配置了 output.library='LibraryName'，则输出和使用的代码如下：
```javascript
// Webpack 输出的代码
exports['LibraryName'] = lib_code;

// 使用库的方法
require('library-name-in-npm')['LibraryName'].doSomething();
其中 library-name-in-npm 是指模块发布到 Npm 代码仓库时的名称。
```
<b>commonjs2</b>
编写的库将通过 CommonJS2 规范导出，输出和使用的代码如下：
```javascript
// Webpack 输出的代码
module.exports = lib_code;

// 使用库的方法
require('library-name-in-npm').doSomething();
CommonJS2 和 CommonJS 规范很相似，差别在于 CommonJS 只能用 exports 导出，而 CommonJS2 在 CommonJS 的基础上增加了 module.exports 的导出方式。

在 output.libraryTarget 为 commonjs2 时，配置 output.library 将没有意义。
```
<b>this</b>
　　编写的库将通过 this 被赋值给通过 library 指定的名称，输出和使用的代码如下：
```javascript
// Webpack 输出的代码
this['LibraryName'] = lib_code;

// 使用库的方法
this.LibraryName.doSomething();
```
<b>window</b>
　　编写的库将通过 window 被赋值给通过 library 指定的名称，即把库挂载到 window 上，输出和使用的代码如下：
```javascript
// Webpack 输出的代码
window['LibraryName'] = lib_code;

// 使用库的方法
window.LibraryName.doSomething();
```
<b>global</b>
```javascript
编写的库将通过 global 被赋值给通过 library 指定的名称，即把库挂载到 global 上，输出和使用的代码如下：

// Webpack 输出的代码
global['LibraryName'] = lib_code;

// 使用库的方法
global.LibraryName.doSomething();
```





