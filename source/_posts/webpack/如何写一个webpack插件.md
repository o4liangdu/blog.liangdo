---
title: 如何写一个webpack插件
date: 2021/04/26 16:48:37
tags: [webpack, nodejs, js]
categories: [webpack, 实战项目]
top: 3
toc: true
---
### 如何写一个webpack插件
上篇文章{% post_link webpack/如何写一个loader %}和{% post_link webpack/如何写一个babel插件 %}中我们基于字符串完成了一个webpack loader，这篇文章将会实现一个webpack插件  

#### 相关原理
webpack插件由以下几个部分组成：
1. 一个具名javaScript函数
2. 在插件函数的 prototype 上定义的一个 apply 方法。
3. 指定一个绑定到 webpack 自身的事件钩子。
4. 处理 webpack 内部实例的特定数据。
5. 功能完成后调用 webpack 提供的回调  

其中，值得注意的是webpack自身的事件钩子，他是基于[tapable](https://zhuanlan.zhihu.com/p/79221553) 实现的,提供了同步与异步两种钩子，下面是基于tapable实现的一个同步钩子：
```js
const { SyncHook } = require('tapable');
const hook = new SyncHook(['name']);
hook.tap('hello', (name) => {
    console.log(`hello ${name}`);
});
hook.call('ali');
// hello ali
```
下面是webpack源码中定义的一些钩子：
```js
class Compiler extends Tapable {
    constructor(context) {
        super();
        this.hooks = {
            /** @type {SyncBailHook<Compilation>} */
            shouldEmit: new SyncBailHook(["compilation"]),
            /** @type {AsyncSeriesHook<Stats>} */
            done: new AsyncSeriesHook(["stats"]),
            /** @type {AsyncSeriesHook<>} */
            additionalPass: new AsyncSeriesHook([]),
            /** @type {AsyncSeriesHook<Compiler>} */
            beforeRun: new AsyncSeriesHook(["compiler"]),
            /** @type {AsyncSeriesHook<Compiler>} */
            run: new AsyncSeriesHook(["compiler"]),
            /** @type {AsyncSeriesHook<Compilation>} */
            emit: new AsyncSeriesHook(["compilation"]),
            /** @type {AsyncSeriesHook<string, Buffer>} */
            assetEmitted: new AsyncSeriesHook(["file", "content"]),
            /** @type {AsyncSeriesHook<Compilation>} */
            afterEmit: new AsyncSeriesHook(["compilation"]),

            /** @type {SyncHook<Compilation, CompilationParams>} */
            thisCompilation: new SyncHook(["compilation", "params"]),
            /** @type {SyncHook<Compilation, CompilationParams>} */
            compilation: new SyncHook(["compilation", "params"]),
            /** @type {SyncHook<NormalModuleFactory>} */
            normalModuleFactory: new SyncHook(["normalModuleFactory"]),
            /** @type {SyncHook<ContextModuleFactory>}  */
            contextModuleFactory: new SyncHook(["contextModulefactory"]),

            /** @type {AsyncSeriesHook<CompilationParams>} */
            beforeCompile: new AsyncSeriesHook(["params"]),
            /** @type {SyncHook<CompilationParams>} */
            compile: new SyncHook(["params"]),
            /** @type {AsyncParallelHook<Compilation>} */
            make: new AsyncParallelHook(["compilation"]),
            /** @type {AsyncSeriesHook<Compilation>} */
            afterCompile: new AsyncSeriesHook(["compilation"]),

            /** @type {AsyncSeriesHook<Compiler>} */
            watchRun: new AsyncSeriesHook(["compiler"]),
            /** @type {SyncHook<Error>} */
            failed: new SyncHook(["error"]),
            /** @type {SyncHook<string, string>} */
            invalid: new SyncHook(["filename", "changeTime"]),
            /** @type {SyncHook} */
            watchClose: new SyncHook([]),

            /** @type {SyncBailHook<string, string, any[]>} */
            infrastructureLog: new SyncBailHook(["origin", "type", "args"]),

            // TODO the following hooks are weirdly located here
            // TODO move them for webpack 5
            /** @type {SyncHook} */
            environment: new SyncHook([]),
            /** @type {SyncHook} */
            afterEnvironment: new SyncHook([]),
            /** @type {SyncHook<Compiler>} */
            afterPlugins: new SyncHook(["compiler"]),
            /** @type {SyncHook<Compiler>} */
            afterResolvers: new SyncHook(["compiler"]),
            /** @type {SyncBailHook<string, Entry>} */
            entryOption: new SyncBailHook(["context", "entry"])
        };
    }}
```
##### Compiler 和 Compilation
compiler 对象代表了完整的 webpack 环境配置。这个对象在启动 webpack 时被一次性建立，并配置好所有可操作的设置，包括 options，loader 和 plugin。当在 webpack 环境中应用一个插件时，插件将收到此 compiler 对象的引用。可以使用它来访问 webpack 的主环境。

compilation 对象代表了一次资源版本构建。当运行 webpack 开发环境中间件时，每当检测到一个文件变化，就会创建一个新的 compilation，从而生成一组新的编译资源。一个 compilation 对象表现了当前的模块资源、编译生成资源、变化的文件、以及被跟踪依赖的状态信息。compilation 对象也提供了很多关键时机的回调，以供插件做自定义处理时选择使用。

我们可以通过操作compilation对象来改变webpack构建的中间资源，从而实现我们需要的功能

#### 实践
[这里](https://github.com/o4liangdu/statistics-webpack-plugin.git)是我实现的一个plugin的项目，主要功能是在webpack打包完成以后生成dist目录各个打包后文件的大小：  
![](http://liangdo-top.oss-cn-shenzhen.aliyuncs.com/blog/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202021-08-23%20175148.png)  

项目结构与{% post_link webpack/如何写一个loader %}类似，主要实现如下：
```js
// 生成一个目录项目目录的文件夹
class FileListPlugin {
    constructor(options) {
        this.options = options
    }
    apply(compiler) {
        compiler.hooks.emit.tap('fileListPlugin', (compilation) => {
            let assets = compilation.assets
            console.log(assets, "111")
            let content = 'In this build:\r\n'
            Object.entries(assets).forEach(([fileName, fileSize]) => {
                content += `--${fileName} —— ${Math.ceil(fileSize.size() / 1024)}kb\r\n`
            })
            console.log('====content====', content)
            assets[this.options.filename] = {
                // 内容存在assets中
                source() {
                    return content
                },
                size() {
                    return content.length
                }
            }
        })
    }
}

module.exports = FileListPlugin
```
重点数据有打印出来，方便配合理解。可以看到，我们遍历取到compilation.assets里面的文件名和大小信息，并存进content中，然后在assets中新增一个我们调用插件时传进来的文件名，这个资源就被加进assets中了，这个插件的任务也就完成了，最后的输出也会输出这个新添加的文件。
当然，webpack还提供了很多钩子，难点在于理解每个钩子中compilation的形式，结合nodejs做灵活的处理，这需要我们在实际工作中总结。
