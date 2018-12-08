---
title: 关于Webpack详述系列文章 （第四篇）
subtitle: wp
date: 2018-6-29
categories: WebPack
tags: 
    - WebPack
---
<b> [WebPack官网](https://webpack-china.org/)</b>

## 1. webpack基本概念
<b>Entry：</b>入口，Webpack 执行构建的第一步将从 Entry 开始，可抽象成输入。
<b>Module：</b>模块，在 Webpack 里一切皆模块，一个模块对应着一个文件。Webpack 会从配置的 Entry 开始递归找出所有依赖的模块。
<b>Chunk：</b>代码块，一个 Chunk 由多个模块组合而成，用于代码合并与分割。 
<b>Loader：</b>模块转换器，用于把模块原内容按照需求转换成新内容。 
<b>Plugin：</b>扩展插件，在 Webpack 构建流程中的特定时机会广播出对应的事件，插件可以监听这些事件的发生，在特定时机做对应的事情。

## 2. 流程概括
+ 初始化参数：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；
+ 开始编译：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译；
+ 确定入口：根据配置中的 entry 找出所有的入口文件；
+ 编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；
+ 完成模块编译：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系；
+ 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
+ 输出完成：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。
>在以上过程中，Webpack 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。

## 3.详细流程
WebPack的构建流程可以分为以下三大阶段：
+ 初始化：启动构建，读取与合并配置参数，加载 Plugin，实例化 Compiler。
+ 编译：从 Entry 发出，针对每个 Module 串行调用对应的 Loader 去翻译文件内容，再找到该 Module 依赖的 Module，递归地进行编译处理。
+ 输出：对编译后的 Module 组合成 Chunk，把 Chunk 转换成文件，输出到文件系统。

#### 3.1 初始化阶段 
事件名|                            解释                            |/
------|----|----|------|------|--------|--------|-------|------|---------|--------|------
初始化参数 |从配置文件和 Shell 语句中读取与合并参数，得出最终的参数。|webpack-cli/bin/webpack.js:436
实例化 Compiler |用上一步得到的参数初始化 Compiler 实例，Compiler 负责文件监听和启动编译。Compiler 实例中包含了完整的 Webpack 配置，全局只有一个 Compiler 实例。| webpack/lib/webpack.js:32
加载插件 | 依次调用插件的 apply 方法，让插件可以监听后续的所有事件节点。同时给插件传入 compiler 实例的引用，以方便插件通过 compiler 调用 Webpack 提供的 API。	| webpack/lib/webpack.js:42
environment	|开始应用 Node.js 风格的文件系统到 compiler 对象，以方便后续的文件寻找和读取。|	webpack/lib/webpack.js:40
entry-option | 读取配置的 Entrys，为每个 Entry 实例化一个对应的 EntryPlugin，为后面该 Entry 的递归解析工作做准备。| webpack/lib/webpack.js:275
after-plugins | 调用完所有内置的和配置的插件的 apply 方法。| webpack/lib/WebpackOptionsApply.js:359
after-resolvers | 根据配置初始化完 resolver，resolver 负责在文件系统中寻找指定路径的文件。	|webpack/lib/WebpackOptionsApply.js:396

#### 3.2 编译阶段
事件名|                            解释                            |/
------|----|----|------|------|--------|--------|-------|------|---------|--------|------
run	| 启动一次新的编译。 | webpack/lib/webpack.js:194
watch-run | 和 run 类似，区别在于它是在监听模式下启动的编译，在这个事件中可以获取到是哪些文件发生了变化导致重新启动一次新的编译。
compile	| 该事件是为了告诉插件一次新的编译将要启动，同时会给插件带上 compiler 对象。| webpack/lib/Compiler.js:455
compilation	| 当 Webpack 以开发模式运行时，每当检测到文件变化，一次新的 Compilation 将被创建。一个 Compilation 对象包含了当前的模块资源、编译生成资源、变化的文件等。Compilation 对象也提供了很多事件回调供插件做扩展。 |	webpack/lib/Compiler.js:418
make | 一个新的 Compilation 创建完毕，即将从 Entry 开始读取文件，根据文件类型和配置的 Loader 对文件进行编译，编译完后再找出该文件依赖的文件，递归的编译和解析。| webpack/lib/Compiler.js:459
after-compile | 一次 Compilation 执行完成。| webpack/lib/Compiler.js:467

在编译阶段中，最重要的要数 compilation 事件了，因为在 compilation 阶段调用了 Loader 完成了每个模块的转换操作，在 compilation 阶段又包括很多小的事件，它们分别是：

事件名|                            解释                            |/
------|----|----|------|------|--------|--------|-------|------|---------|--------|------
should-emit |所有需要输出的文件已经生成好，询问插件哪些文件需要输出，哪些不需要。| webpack/lib/Compiler.js:146
emit | 确定好要输出哪些文件后，执行文件输出，可以在这里获取和修改输出内容。	| webpack/lib/Compiler.js:287
after-emit | 文件输出完毕。	| webpack/lib/Compiler.js:278
done | 成功完成一次完成的编译和输出流程。| webpack/lib/Compiler.js:166
failed | 如果在编译和输出流程中遇到异常导致 Webpack 退出时，就会直接跳转到本步骤，插件可以在本事件中获取到具体的错误原因。
> 在输出阶段已经得到了各个模块经过转换后的结果和其依赖关系，并且把相关模块组合在一起形成一个个 Chunk。 在输出阶段会根据 Chunk 的类型，使用对应的模版生成最终要要输出的文件内容。

## 4. 代码流程

#### 4.1 node_modules/.bin/webpack-cli
+ 通过yargs获得shell中的参数
+ 把webpack.config.js中的参数和shell参数整合到options对象上
+ 调用webpack-cli/bin/webpack.js开始编译和打包
+ webpack-cli/bin/webpack.js中返回一个compiler对象，并调用了compiler.run()
+ lib/Compiler.js中，run方法触发了before-run、run两个事件，然后通过readRecords读取文件，通过compile进行打包,该方法中实例化了一个Compilation类
+ 打包时触发before-compile、compile、make等事件
+ make事件会触发SingleEntryPlugin监听函数，调用compilation.addEntry方法
+ 这个方法通过调用其私有方法_addModuleChain完成了两件事：根据模块的类型获取对应的模块工厂并创建模块；构建模块
    + 调用loader处理模块之间的依赖
    + 将loader处理后的文件通过acorn抽象成抽象语法树AST
    + 遍历AST，构建该模块的所有依赖
+ 调用Compilation的finish及seal方法

#### 4.2 关键事件
+ entry-option：初始化options
+ after-plugins
+ after-resolvers
+ environment
+ after-environment
+ before-run
+ run：开始编译
+ watch-run
+ normal-module-factory
+ context-module-factory
+ before-compile
+ compile
+ this-compilation
+ compilation
+ make：从entry开始递归分析依赖并对依赖进行build
+ build-module：使用loader加载文件并build模块
+ normal-module-loader：对loader加载的文件用acorn编译，生成抽象语法树AST
+ program：开始对AST进行遍历，当遇到require时触发call require事件
+ after-compile
+ should-emit
+ need-additional-pass
+ seal：所有依赖build完成，开始对chunk进行优化（抽取公共模块、加hash等）
+ optimize-chunk-assets：优化代码
+ emit：把各个chunk输出到结果文件
+ after-emit
+ done
+ failed
+ invalid
+ watch-close

## 参考
[plugins](https://github.com/webpack/docs/wiki/plugins)
[how to write a plugins](https://github.com/webpack/docs/wiki/how-to-write-a-plugin)
[Webpack编写一个插件](https://www.webpackjs.com/contribute/writing-a-plugin/)
[Webpack编写一个loader](https://www.webpackjs.com/contribute/writing-a-loader/)



















