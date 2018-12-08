---
title: 关于Webpack详述系列文章 （第三篇）
subtitle: wp
date: 2018-6-7
categories: WebPack
tags: 
    - WebPack
---
<b> [WebPack官网](https://webpack-china.org/)</b>

##  类图 
<img src="https://images2018.cnblogs.com/blog/1140938/201806/1140938-20180611104615852-1523780791.png"/>

## 1. 模块
Module是webpack中最核心的类，要加载定的一切和依赖都是Module. 它有很多子类
+ RawModule
+ NormalModule
+ MultiModule
+ ContextModule
+ DelegatedModule
+ DllModule
+ ExternalModule

## 2. 依赖 
+ Module类继承自DependenciesBlock，它有一个dependencies数组，表示此模块依赖的其他模块。
+ webpack使用Dependency的各种子类来表示不同的模块加载规范，每个规范都有自己的DependencyParserPlugin来加载
    + AMDRequireDependency
    + AMDDefineDependency
    + AMDRequireArrayDependency
    + CommonJsRequireDependency
    + SystemImportDependency
+ 每个Dependency都有自己的Template方法，用来生成加载该依赖模块定的JS代码
+ 不同的模块工厂可以生产不同的模块

## 3.执行流程 
<img src="https://images2018.cnblogs.com/blog/1140938/201806/1140938-20180611104701548-793450873.png"/>
<img src="https://images2018.cnblogs.com/blog/1140938/201806/1140938-20180611104701548-793450873.png"/>
+ compile 开始编译
+ make 从入口点分析模块及其依赖的模块，创建这些模块对象
+ build-module 构建模块
+ after-compile 完成构建
+ seal 封装构建结果
+ emit 把各个chunk输出到结果文件
+ after-emit 完成输出

#### 3.1 编译核心对象 Compilation
```javascript
this.compiler = compiler;       //Compiler对象
this.resolvers = compiler.resolvers;   //模块解析器
this.mainTemplate = new MainTemplate(this.outputOptions);   //生成主模块JS
this.chunkTemplate = new ChunkTemplate(this.outputOptions, this.mainTemplate);//异步加载的模块JS
this.hotUpdateChunkTemplate = new HotUpdateChunkTemplate(this.outputOptions);//热更新是的代码块模版JS
this.moduleTemplate = new ModuleTemplate(this.outputOptions); //入口模块代码
this.entries = []; //入口
this.preparedChunks = [];//预先加载的chunk
this.chunks = [];//所有的chunk
this.namedChunks = {};//每个都对应一个名字，可以通过namedChunks[name]获取chunk
this.modules = []; //所有module
this.assets = {}; //保存所有生成的文件
this.children = []; // 保存子Compilation对象，子Compilation对象依赖它的上级Compilation对象生成的结果，所以要等父Compilation编译完成才能开始。
this.dependencyFactories = new ArrayMap();//保存Dependency和ModuleFactory的对应关系，方便创建该依赖对应的Module
this.dependencyTemplates = new ArrayMap();   //保存Dependency和Template对应关系，方便生成加载此模块的代码
```

#### 3.2 开始模块编译
+ SingleEntryPlugin,MultiEntryPlugin两个插件中注册了对make事件的监听，当Compiler执行make时，触发对 Compilation.addEntry方法的调用. 在addEntry方法内调用私有方法_addModuleChain
+ 使用acorn生成AST，并遍历AST收集依赖
+ webpack使用acorn解析每一个经loader处理过的source，并且成AST，然后遍历所有节点，当遇到require调用时，会分析是AMD的还是CMD的调用，或者是require.ensure.
+ webpack在build模块时 (调用doBuild方法)，要先调用相应的loader对source进行加工，生成一段JS代码后交给acorn解析生成AST.所以不管是css文件，还是jpg文件，还是html模版，最终经过loader处理会变成一个module
+ 比如url-loader，根据loader配置生成一段dataURL或者使用调用loadercontext的emitFile方法向assets添加一个文件
```javascript
_addModuleChain(context, dependency, onModule, callback) {
  //根据依赖模块的类型获取对应的模块工厂，用于后边创建模块。
  const moduleFactory = this.dependencyFactories.get(dependency.constructor);
  //使用模块工厂创建模块，并将创建出来的module作为参数传给回调方法:就是下边`function(err, module)`的参数
   moduleFactory.create((err, module) => {
       const addModuleResult = this.addModule(module);
       //下面要对module进行build了。包括调用loader处理源文件，使用acorn生成AST，将遍历AST,遇到requirt等依赖时，创建依赖(Dependency)加入依赖数组
       this.buildModule(module, false, null, null, err => {
           //module已经build完了，依赖也收集好了，开始处理依赖的module
          this.processModuleDependencies(module, err => {
              callback(null, module);
          });
       });
   })
}
```

#### 3.3 开始封装
+ 调用seal方法封装，要逐次对每个module和chunk进行整理，生成编译后的源码，合并，拆分，生成hash
+ webpack会根据不同的插件，如MinChunkSizePlugin,LimitChunkCountPlugin 将不同的module整理到不同的chunk里，每个chunk最终对应一个输出文件
+ 此时所有的module仍然保存的是编译前的 原始文件内容。webpack会把源代码里的reuqire调用换成webpack模块加载代码,也就是最终编译后的代码

#### 3.4 通过Template生成结果代码
调用compilation类里的createChunkAssets方法
```javascript
//如果是入口，则使用MainTemplate生成结果，否则使用ChunkTemplate
const template = chunk.hasRuntime()
                    ? this.mainTemplate
                    : this.chunkTemplate;
在MainTemplate和ChunkTemplate需要根据依赖的模块，逐个调用ModuleTemplate的render方法

render(module, dependencyTemplates, options) {
    //生成该模块结果代码的方法,source是一个抽象方法，在Module的不同子类里会重写该方法
    const moduleSource = module.source(
            dependencyTemplates,
            this.runtimeTemplate
        );
}
```
在子类NormalModule的source方法里，必须把源代码中的require()引入的模块代码替换成webpack的模块加载代码 NormalModule.js
```javascript
source(dependencyTemplates, runtimeTemplate) {
    const source = this.generator.generate(
            this,
            dependencyTemplates,
            runtimeTemplate
        );
}
class JavascriptGenerator {
    //source是一个ReplaceSource,可利用dep参数的range属性定位require调用在源码中的位置，从而实现替换
    //range: 根据paser:acorn的文档说明，保存了AST节点在源码中的起始位置和结束位置[ start , end ]
    //export default 'hello'; 会转换为
    //"/* harmony default export */ var __WEBPACK_MODULE_DEFAULT_EXPORT__ = ('hello');"
    generate(module, dependencyTemplates, runtimeTemplate) {
        const source = new ReplaceSource(originalSource);
    }
}
```

#### 3.5 输出到结果文件
webpack会在Compiler的emitAssets方法里把compilation.assets里的结果写到输出文件里，在此前会先创建输出目录。 所以当你要开发一些自定义的插件要输出一些结果时，把文件放入compilation.assets里即可。

#### 3.6 调试
```javascript
npm i vscode-webpack-debugger -D
```

#### 3.7 日志

###### 3.7.1 注册流程
```javascript
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap Resolver: before/after
tap Resolver: step hooks
tapAsync ParsePlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync { stage: 10, name: 'ModuleKindPlugin' }
tapAsync { stage: 10, name: 'JoinRequestPlugin' }
tapAsync TryNextPlugin
tapAsync ModulesInHierachicDirectoriesPlugin
tapAsync FileKindPlugin
tapAsync DirectoryExistsPlugin
tapAsync MainFieldPlugin
tapAsync UseFilePlugin
tapAsync AppendPlugin
tapAsync SymlinkPlugin
tapAsync FileExistsPlugin
tapAsync NextPlugin
tapAsync ResultPlugin
tapAsync ModuleAppendPlugin
tap Compiler
tap ResolverFactory
tap NodeEnvironmentPlugin
tap JsonpTemplatePlugin
tap FetchCompileWasmTemplatePlugin
tap FunctionModulePlugin
tap NodeSourcePlugin
tap LoaderTargetPlugin
tap JavascriptModulesPlugin
tap JsonModulesPlugin
tap WebAssemblyModulesPlugin
tap EntryOptionPlugin
tap SingleEntryPlugin
tapAsync SingleEntryPlugin
tap CompatibilityPlugin
tap HarmonyModulesPlugin
tap AMDPlugin
tap CommonJsPlugin
tap LoaderPlugin
tap NodeStuffPlugin
tap RequireJsStuffPlugin
tap APIPlugin
tap ConstPlugin
tap UseStrictPlugin
tap RequireIncludePlugin
tap RequireEnsurePlugin
tap RequireContextPlugin
tap ImportPlugin
tap SystemPlugin
tap WarnNoModeSetPlugin
tap EnsureChunkConditionsPlugin
tap RemoveParentModulesPlugin
tap RemoveEmptyChunksPlugin
tap MergeDuplicateChunksPlugin
tap FlagIncludedChunksPlugin
tap OccurrenceOrderPlugin
tap SideEffectsFlagPlugin
tap FlagDependencyExportsPlugin
tap FlagDependencyUsagePlugin
tap ModuleConcatenationPlugin
tap SplitChunksPlugin
tap NoEmitOnErrorsPlugin
tap DefinePlugin
tap { name: 'UglifyJSPlugin' }
tap SizeLimitsPlugin
tap TemplatedPathPlugin
tap RecordIdsPlugin
tap WarnCaseSensitiveModulesPlugin
tap WebpackOptionsApply
Compiler:before-run
Compiler:run
tap NormalModuleFactory
normal-module-factory
tap ContextModuleFactory
Compiler:context-module-factory
Compiler:before-compile
Compiler:compile
tap Compilation
tap MainTemplate
Compiler:this-compilation
tap JsonpMainTemplatePlugin
tap JsonpChunkTemplatePlugin
tap JsonpHotUpdateChunkTemplatePlugin
tap FetchCompileWasmMainTemplatePlugin
tap WasmModuleTemplatePlugin
Compiler:compilation
tap FunctionModuleTemplatePlugin
tapAsync { name: 'UglifyJSPlugin' }
tapAsync UnsafeCachePlugin
    ResolverFactory:resolver
tapAsync AliasFieldPlugin
tapAsync AliasPlugin
    Resolver:resolveStep
tap Parser
    NormalModuleFactory:parser
tap HarmonyDetectionParserPlugin
tap HarmonyImportDependencyParserPlugin
tap HarmonyExportDependencyParserPlugin
tap HarmonyTopLevelThisParserPlugin
tap AMDRequireDependenciesBlockParserPlugin
tap AMDDefineDependencyParserPlugin
tap CommonJsRequireDependencyParserPlugin
tap RequireResolveDependencyParserPlugin
tap RequireIncludeDependencyParserPlugin
tap RequireEnsureDependenciesBlockParserPlugin
tap RequireContextDependencyParserPlugin
tap ImportParserPlugin
```

###### 3.7.2 触发流程
```javascript
Compiler:before-run
Compiler:run
normal-module-factory
Compiler:context-module-factory
Compiler:before-compile
Compiler:compile
Compiler:this-compilation
Compiler:compilation
Compiler:make
    NormalModuleFactory:before-resolve
    NormalModuleFactory:factory
    NormalModuleFactory:resolver
    ResolverFactory:resolve-options
    ResolverFactory:resolver
    Resolver:resolveStep
    NormalModuleFactory:parser
    NormalModuleFactory:generator
    NormalModuleFactory:after-resolve
    NormalModuleFactory:create-module
    NormalModuleFactory:module
  Compilation:build-module
  Compilation:succeed-module
  Compilation:finish-modules
  Compilation:seal
  Compilation:optimize-dependencies-basic
  Compilation:optimize-dependencies
  Compilation:optimize-dependencies-advanced
  Compilation:after-optimize-dependencies
  Compilation:optimize
  Compilation:optimize-modules-basic
  Compilation:optimize-modules
  Compilation:optimize-modules-advanced
  Compilation:after-optimize-modules
  Compilation:optimize-chunks-basic
  Compilation:optimize-chunks
  Compilation:optimize-chunks-advanced
  Compilation:after-optimizeChunks
  Compilation:optimize-tree
  Compilation:after-optimize-tree
  Compilation:optimize-chunk-modules-basic
  Compilation:optimize-chunk-modules
  Compilation:optimize-chunk-modules-advanced
  Compilation:after-optimize-chunk-modules
  Compilation:should-record
  Compilation:revive-modules
  Compilation:optimize-moduleOrder
  Compilation:advanced-optimize-module-order
  Compilation:before-moduleIds
  Compilation:module-ids
  Compilation:optimize-module-ids
  Compilation:after-optimize-module-ids
  Compilation:revive-chunks
  Compilation:optimize-chunk-order
  Compilation:before-chunk-ids
  Compilation:optimize-chunk-ids
  Compilation:after-optimize-chunk-ids
  Compilation:record-modules
  Compilation:record-chunks
  Compilation:before-hash
    MainTemplate:hash
    ChunkTemplate:hash
    ModuleTemplate:hash
    MainTemplate:hash-for-chunk
  Compilation:chunk-hash
  Compilation:after-hash
  Compilation:record-hash
  Compilation:before-module-assets
  Compilation:should-generate-chunk-assets
  Compilation:before-chunk-assets
    MainTemplate:render-manifest
    MainTemplate:global-hash-paths
    MainTemplate:bootstrap
    MainTemplate:localVars
    MainTemplate:require
    MainTemplate:require-extensions
    MainTemplate:asset-path
    MainTemplate:before-startup
    MainTemplate:startup
    ModuleTemplate:content
    ModuleTemplate:module
    ModuleTemplate:render
    ModuleTemplate:package
    MainTemplate:render-with-entry
  Compilation:chunk-asset
  Compilation:additional-chunk-assets
  Compilation:record
  Compilation:additional-assets
  Compilation:optimize-chunk-assets
  Compilation:after-optimize-chunk-assets
  Compilation:optimize-assets
  Compilation:after-optimize-assets
  Compilation:need-additional-seal
  Compilation:after-seal
Compiler:after-compile
Compiler:should-emit
Compiler:emit
Compiler:after-emit
Compiler:done
```

###### 3.7.3 完整流程
```javascript
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap Resolver: before/after
tap Resolver: step hooks
tapAsync ParsePlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync { stage: 10, name: 'ModuleKindPlugin' }
tapAsync { stage: 10, name: 'JoinRequestPlugin' }
tapAsync TryNextPlugin
tapAsync ModulesInHierachicDirectoriesPlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync FileKindPlugin
tapAsync TryNextPlugin
tapAsync DirectoryExistsPlugin
tapAsync MainFieldPlugin
tapAsync UseFilePlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync TryNextPlugin
tapAsync AppendPlugin
tapAsync AppendPlugin
tapAsync AppendPlugin
tapAsync SymlinkPlugin
tapAsync FileExistsPlugin
tapAsync NextPlugin
tapAsync ResultPlugin
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap Resolver: before/after
tap Resolver: step hooks
tapAsync ParsePlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync { stage: 10, name: 'ModuleKindPlugin' }
tapAsync { stage: 10, name: 'JoinRequestPlugin' }
tapAsync TryNextPlugin
tapAsync ModulesInHierachicDirectoriesPlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync FileKindPlugin
tapAsync TryNextPlugin
tapAsync DirectoryExistsPlugin
tapAsync MainFieldPlugin
tapAsync UseFilePlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync TryNextPlugin
tapAsync AppendPlugin
tapAsync AppendPlugin
tapAsync AppendPlugin
tapAsync SymlinkPlugin
tapAsync FileExistsPlugin
tapAsync NextPlugin
tapAsync ResultPlugin
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap Resolver: before/after
tap Resolver: step hooks
tapAsync ParsePlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync { stage: 10, name: 'ModuleKindPlugin' }
tapAsync { stage: 10, name: 'JoinRequestPlugin' }
tapAsync TryNextPlugin
tapAsync ModulesInHierachicDirectoriesPlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync FileKindPlugin
tapAsync TryNextPlugin
tapAsync DirectoryExistsPlugin
tapAsync NextPlugin
tapAsync ResultPlugin
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap Resolver: before/after
tap Resolver: step hooks
tapAsync ParsePlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync { stage: 10, name: 'ModuleKindPlugin' }
tapAsync { stage: 10, name: 'JoinRequestPlugin' }
tapAsync TryNextPlugin
tapAsync ModulesInHierachicDirectoriesPlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync FileKindPlugin
tapAsync TryNextPlugin
tapAsync DirectoryExistsPlugin
tapAsync NextPlugin
tapAsync ResultPlugin
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap Resolver: before/after
tap Resolver: step hooks
tapAsync ParsePlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync { stage: 10, name: 'ModuleKindPlugin' }
tapAsync { stage: 10, name: 'JoinRequestPlugin' }
tapAsync ModuleAppendPlugin
tapAsync TryNextPlugin
tapAsync ModulesInHierachicDirectoriesPlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync FileKindPlugin
tapAsync TryNextPlugin
tapAsync DirectoryExistsPlugin
tapAsync MainFieldPlugin
tapAsync MainFieldPlugin
tapAsync UseFilePlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync TryNextPlugin
tapAsync AppendPlugin
tapAsync AppendPlugin
tapAsync AppendPlugin
tapAsync SymlinkPlugin
tapAsync FileExistsPlugin
tapAsync NextPlugin
tapAsync ResultPlugin
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap Resolver: before/after
tap Resolver: step hooks
tapAsync ParsePlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync { stage: 10, name: 'ModuleKindPlugin' }
tapAsync { stage: 10, name: 'JoinRequestPlugin' }
tapAsync ModuleAppendPlugin
tapAsync TryNextPlugin
tapAsync ModulesInHierachicDirectoriesPlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync FileKindPlugin
tapAsync TryNextPlugin
tapAsync DirectoryExistsPlugin
tapAsync MainFieldPlugin
tapAsync MainFieldPlugin
tapAsync UseFilePlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync TryNextPlugin
tapAsync AppendPlugin
tapAsync AppendPlugin
tapAsync AppendPlugin
tapAsync SymlinkPlugin
tapAsync FileExistsPlugin
tapAsync NextPlugin
tapAsync ResultPlugin
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap Compiler
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap ResolverFactory
tap NodeEnvironmentPlugin
tap JsonpTemplatePlugin
tap FetchCompileWasmTemplatePlugin
tap FunctionModulePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap LoaderTargetPlugin
tap JavascriptModulesPlugin
tap JsonModulesPlugin
tap WebAssemblyModulesPlugin
tap EntryOptionPlugin
tap SingleEntryPlugin
tapAsync SingleEntryPlugin
tap CompatibilityPlugin
tap HarmonyModulesPlugin
tap AMDPlugin
tap AMDPlugin
tap CommonJsPlugin
tap LoaderPlugin
tap LoaderPlugin
tap NodeStuffPlugin
tap RequireJsStuffPlugin
tap APIPlugin
tap ConstPlugin
tap UseStrictPlugin
tap RequireIncludePlugin
tap RequireEnsurePlugin
tap RequireContextPlugin
tap ImportPlugin
tap SystemPlugin
tap WarnNoModeSetPlugin
tap EnsureChunkConditionsPlugin
tap RemoveParentModulesPlugin
tap RemoveEmptyChunksPlugin
tap MergeDuplicateChunksPlugin
tap FlagIncludedChunksPlugin
tap OccurrenceOrderPlugin
tap SideEffectsFlagPlugin
tap SideEffectsFlagPlugin
tap FlagDependencyExportsPlugin
tap FlagDependencyUsagePlugin
tap ModuleConcatenationPlugin
tap SplitChunksPlugin
tap NoEmitOnErrorsPlugin
tap NoEmitOnErrorsPlugin
tap DefinePlugin
tap { name: 'UglifyJSPlugin' }
tap SizeLimitsPlugin
tap TemplatedPathPlugin
tap RecordIdsPlugin
tap WarnCaseSensitiveModulesPlugin
tap WebpackOptionsApply
tap WebpackOptionsApply
tap WebpackOptionsApply
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap AMDPlugin
Compiler:before-run
Compiler:run
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap NormalModuleFactory
tap NormalModuleFactory
tap NormalModuleFactory
normal-module-factory
tap SideEffectsFlagPlugin
tap SideEffectsFlagPlugin
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap ContextModuleFactory
Compiler:context-module-factory
Compiler:before-compile
Compiler:compile
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap Compilation
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap MainTemplate
tap MainTemplate
tap MainTemplate
tap MainTemplate
tap MainTemplate
tap MainTemplate
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
Compiler:this-compilation
tap JsonpMainTemplatePlugin
tap JsonpMainTemplatePlugin
tap JsonpMainTemplatePlugin
tap JsonpMainTemplatePlugin
tap JsonpMainTemplatePlugin
tap JsonpMainTemplatePlugin
tap JsonpMainTemplatePlugin
tap JsonpMainTemplatePlugin
tap JsonpMainTemplatePlugin
tap JsonpChunkTemplatePlugin
tap JsonpChunkTemplatePlugin
tap JsonpHotUpdateChunkTemplatePlugin
tap JsonpHotUpdateChunkTemplatePlugin
tap FetchCompileWasmMainTemplatePlugin
tap FetchCompileWasmMainTemplatePlugin
tap FetchCompileWasmMainTemplatePlugin
tap FetchCompileWasmMainTemplatePlugin
tap WasmModuleTemplatePlugin
tap WasmModuleTemplatePlugin
tap SplitChunksPlugin
tap SplitChunksPlugin
Compiler:compilation
tap FunctionModuleTemplatePlugin
tap FunctionModuleTemplatePlugin
tap FunctionModuleTemplatePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap LoaderTargetPlugin
tap JavascriptModulesPlugin
tap JavascriptModulesPlugin
tap JavascriptModulesPlugin
tap JavascriptModulesPlugin
tap JavascriptModulesPlugin
tap JavascriptModulesPlugin
tap JavascriptModulesPlugin
tap JavascriptModulesPlugin
tap JavascriptModulesPlugin
tap JsonModulesPlugin
tap JsonModulesPlugin
tap WebAssemblyModulesPlugin
tap WebAssemblyModulesPlugin
tap WebAssemblyModulesPlugin
tap CompatibilityPlugin
tap HarmonyModulesPlugin
tap HarmonyModulesPlugin
tap AMDPlugin
tap AMDPlugin
tap CommonJsPlugin
tap CommonJsPlugin
tap LoaderPlugin
tap NodeStuffPlugin
tap NodeStuffPlugin
tap RequireJsStuffPlugin
tap RequireJsStuffPlugin
tap APIPlugin
tap APIPlugin
tap APIPlugin
tap ConstPlugin
tap ConstPlugin
tap ConstPlugin
tap UseStrictPlugin
tap UseStrictPlugin
tap UseStrictPlugin
tap RequireIncludePlugin
tap RequireIncludePlugin
tap RequireEnsurePlugin
tap RequireEnsurePlugin
tap RequireContextPlugin
tap RequireContextPlugin
tap RequireContextPlugin
tap RequireContextPlugin
tap RequireContextPlugin
tap ImportPlugin
tap ImportPlugin
tap ImportPlugin
tap SystemPlugin
tap SystemPlugin
tap EnsureChunkConditionsPlugin
tap EnsureChunkConditionsPlugin
tap RemoveParentModulesPlugin
tap RemoveParentModulesPlugin
tap RemoveEmptyChunksPlugin
tap RemoveEmptyChunksPlugin
tap MergeDuplicateChunksPlugin
tap FlagIncludedChunksPlugin
tap OccurrenceOrderPlugin
tap OccurrenceOrderPlugin
tap SideEffectsFlagPlugin
tap FlagDependencyExportsPlugin
tap FlagDependencyExportsPlugin
tap FlagDependencyExportsPlugin
tap FlagDependencyUsagePlugin
tap ModuleConcatenationPlugin
tap ModuleConcatenationPlugin
tap ModuleConcatenationPlugin
tap ModuleConcatenationPlugin
tap NoEmitOnErrorsPlugin
tap DefinePlugin
tap DefinePlugin
tap DefinePlugin
tapAsync { name: 'UglifyJSPlugin' }
tap TemplatedPathPlugin
tap TemplatedPathPlugin
tap TemplatedPathPlugin
tap RecordIdsPlugin
tap RecordIdsPlugin
tap RecordIdsPlugin
tap RecordIdsPlugin
tap WarnCaseSensitiveModulesPlugin
Compiler:make
    NormalModuleFactory:before-resolve
    NormalModuleFactory:factory
    NormalModuleFactory:resolver
    ResolverFactory:resolve-options
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap Resolver: before/after
tap Resolver: step hooks
tapAsync UnsafeCachePlugin
tapAsync ParsePlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync { stage: 10, name: 'ModuleKindPlugin' }
tapAsync { stage: 10, name: 'JoinRequestPlugin' }
tapAsync TryNextPlugin
tapAsync ModulesInHierachicDirectoriesPlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync FileKindPlugin
tapAsync TryNextPlugin
tapAsync DirectoryExistsPlugin
tapAsync MainFieldPlugin
tapAsync MainFieldPlugin
tapAsync UseFilePlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync TryNextPlugin
tapAsync AppendPlugin
tapAsync AppendPlugin
tapAsync SymlinkPlugin
tapAsync FileExistsPlugin
tapAsync NextPlugin
tapAsync ResultPlugin
    ResolverFactory:resolver
    ResolverFactory:resolve-options
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap Resolver: before/after
tap Resolver: step hooks
tapAsync UnsafeCachePlugin
tapAsync ParsePlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync AliasFieldPlugin
tapAsync { stage: 10, name: 'ModuleKindPlugin' }
tapAsync { stage: 10, name: 'JoinRequestPlugin' }
tapAsync TryNextPlugin
tapAsync ModulesInHierachicDirectoriesPlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync FileKindPlugin
tapAsync TryNextPlugin
tapAsync DirectoryExistsPlugin
tapAsync MainFieldPlugin
tapAsync MainFieldPlugin
tapAsync MainFieldPlugin
tapAsync UseFilePlugin
tapAsync DescriptionFilePlugin
tapAsync { stage: 10, name: 'NextPlugin' }
tapAsync TryNextPlugin
tapAsync AppendPlugin
tapAsync AppendPlugin
tapAsync AppendPlugin
tapAsync AppendPlugin
tapAsync AliasFieldPlugin
tapAsync SymlinkPlugin
tapAsync FileExistsPlugin
tapAsync NextPlugin
tapAsync ResultPlugin
    ResolverFactory:resolver
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
tapAsync AliasPlugin
    Resolver:resolveStep
    Resolver:resolveStep
    Resolver:resolveStep
    Resolver:resolveStep
    Resolver:resolveStep
    Resolver:resolveStep
    Resolver:resolveStep
    Resolver:resolveStep
    Resolver:resolveStep
    Resolver:resolveStep
tap { name: 'Tapable camelCase', stage: 100 }
tap { name: 'Tapable this.hooks', stage: 200 }
tap Parser
tap Parser
tap Parser
tap Parser
tap Parser
tap Parser
tap Parser
tap Parser
tap Parser
tap Parser
tap Parser
tap Parser
tap Parser
tap Parser
tap Parser
tap Parser
tap Parser
tap Parser
tap Parser
    NormalModuleFactory:parser
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap NodeSourcePlugin
tap CompatibilityPlugin
tap HarmonyDetectionParserPlugin
tap HarmonyDetectionParserPlugin
tap HarmonyDetectionParserPlugin
tap HarmonyDetectionParserPlugin
tap HarmonyDetectionParserPlugin
tap HarmonyDetectionParserPlugin
tap HarmonyDetectionParserPlugin
tap HarmonyDetectionParserPlugin
tap HarmonyDetectionParserPlugin
tap HarmonyDetectionParserPlugin
tap HarmonyDetectionParserPlugin
tap HarmonyImportDependencyParserPlugin
tap HarmonyImportDependencyParserPlugin
tap HarmonyImportDependencyParserPlugin
tap HarmonyImportDependencyParserPlugin
tap HarmonyImportDependencyParserPlugin
tap HarmonyImportDependencyParserPlugin
tap HarmonyImportDependencyParserPlugin
tap HarmonyExportDependencyParserPlugin
tap HarmonyExportDependencyParserPlugin
tap HarmonyExportDependencyParserPlugin
tap HarmonyExportDependencyParserPlugin
tap HarmonyExportDependencyParserPlugin
tap HarmonyExportDependencyParserPlugin
tap HarmonyTopLevelThisParserPlugin
tap AMDRequireDependenciesBlockParserPlugin
tap AMDDefineDependencyParserPlugin
tap AMDPlugin
tap AMDPlugin
tap AMDPlugin
tap AMDPlugin
tap AMDPlugin
tap AMDPlugin
tap AMDPlugin
tap AMDPlugin
tap AMDPlugin
tap AMDPlugin
tap AMDPlugin
tap AMDPlugin
tap AMDPlugin
tap AMDPlugin
tap CommonJsPlugin
tap CommonJsPlugin
tap CommonJsPlugin
tap CommonJsPlugin
tap CommonJsPlugin
tap CommonJsPlugin
tap CommonJsPlugin
tap CommonJsPlugin
tap CommonJsPlugin
tap CommonJsPlugin
tap CommonJsPlugin
tap CommonJsPlugin
tap CommonJsPlugin
tap CommonJsPlugin
tap CommonJsPlugin
tap CommonJsRequireDependencyParserPlugin
tap CommonJsRequireDependencyParserPlugin
tap CommonJsRequireDependencyParserPlugin
tap RequireResolveDependencyParserPlugin
tap RequireResolveDependencyParserPlugin
tap NodeStuffPlugin
tap NodeStuffPlugin
tap NodeStuffPlugin
tap NodeStuffPlugin
tap NodeStuffPlugin
tap NodeStuffPlugin
tap NodeStuffPlugin
tap NodeStuffPlugin
tap NodeStuffPlugin
tap NodeStuffPlugin
tap NodeStuffPlugin
tap RequireJsStuffPlugin
tap RequireJsStuffPlugin
tap RequireJsStuffPlugin
tap RequireJsStuffPlugin
tap APIPlugin
tap APIPlugin
tap APIPlugin
tap APIPlugin
tap APIPlugin
tap APIPlugin
tap APIPlugin
tap APIPlugin
tap APIPlugin
tap APIPlugin
tap APIPlugin
tap APIPlugin
tap APIPlugin
tap APIPlugin
tap ConstPlugin
tap ConstPlugin
tap ConstPlugin
tap ConstPlugin
tap UseStrictPlugin
tap RequireIncludeDependencyParserPlugin
tap RequireIncludePlugin
tap RequireIncludePlugin
tap RequireEnsureDependenciesBlockParserPlugin
tap RequireEnsurePlugin
tap RequireEnsurePlugin
tap RequireContextDependencyParserPlugin
tap ImportParserPlugin
tap SystemPlugin
tap SystemPlugin
tap SystemPlugin
tap SystemPlugin
tap SystemPlugin
tap SystemPlugin
tap SystemPlugin
tap SystemPlugin
tap SystemPlugin
tap SystemPlugin
tap SystemPlugin
tap SystemPlugin
tap ModuleConcatenationPlugin
tap DefinePlugin
tap DefinePlugin
tap DefinePlugin
tap DefinePlugin
tap DefinePlugin
tap DefinePlugin
tap DefinePlugin
    NormalModuleFactory:generator
    NormalModuleFactory:after-resolve
    NormalModuleFactory:create-module
    NormalModuleFactory:module
  Compilation:build-module
  Compilation:succeed-module
  Compilation:finish-modules
  Compilation:seal
  Compilation:optimize-dependencies-basic
  Compilation:optimize-dependencies
  Compilation:optimize-dependencies-advanced
  Compilation:after-optimize-dependencies
  Compilation:optimize
  Compilation:optimize-modules-basic
  Compilation:optimize-modules
  Compilation:optimize-modules-advanced
  Compilation:after-optimize-modules
  Compilation:optimize-chunks-basic
  Compilation:optimize-chunks
  Compilation:optimize-chunks-advanced
  Compilation:after-optimizeChunks
  Compilation:optimize-tree
  Compilation:after-optimize-tree
  Compilation:optimize-chunk-modules-basic
  Compilation:optimize-chunk-modules
  Compilation:optimize-chunk-modules-advanced
  Compilation:after-optimize-chunk-modules
  Compilation:should-record
  Compilation:revive-modules
  Compilation:optimize-moduleOrder
  Compilation:advanced-optimize-module-order
  Compilation:before-moduleIds
  Compilation:module-ids
  Compilation:optimize-module-ids
  Compilation:after-optimize-module-ids
  Compilation:revive-chunks
  Compilation:optimize-chunk-order
  Compilation:before-chunk-ids
  Compilation:optimize-chunk-ids
  Compilation:after-optimize-chunk-ids
  Compilation:record-modules
  Compilation:record-chunks
  Compilation:before-hash
    MainTemplate:hash
    ChunkTemplate:hash
    ModuleTemplate:hash
    ModuleTemplate:hash
    MainTemplate:hash
    MainTemplate:hash-for-chunk
  Compilation:chunk-hash
  Compilation:after-hash
  Compilation:record-hash
  Compilation:before-module-assets
  Compilation:should-generate-chunk-assets
  Compilation:before-chunk-assets
    MainTemplate:render-manifest
    MainTemplate:global-hash-paths
    MainTemplate:bootstrap
    MainTemplate:localVars
    MainTemplate:require
    MainTemplate:require-extensions
    MainTemplate:asset-path
    MainTemplate:before-startup
    MainTemplate:startup
    ModuleTemplate:content
    ModuleTemplate:module
    ModuleTemplate:render
    ModuleTemplate:package
    MainTemplate:render-with-entry
    MainTemplate:asset-path
  Compilation:chunk-asset
  Compilation:additional-chunk-assets
  Compilation:record
  Compilation:additional-assets
  Compilation:optimize-chunk-assets
  Compilation:after-optimize-chunk-assets
  Compilation:optimize-assets
  Compilation:after-optimize-assets
  Compilation:need-additional-seal
  Compilation:after-seal
Compiler:after-compile
Compiler:should-emit
Compiler:emit
    MainTemplate:asset-path
Compiler:after-emit
Compiler:done
```

## 版本
+ webpack 4.2.0
+ webpack-cli 2.0.13
+ vscode-webpack-debugger


