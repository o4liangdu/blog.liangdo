---
title: babel解剖
date: 2021/05/05 13:38:31
tags: [webpack, babel, js, 前端架构]
categories: [webpack, babel]
top: 3
toc: true
---
### babel解剖
当聊到Babel的作用，很多人第一反应是：用来实现API polyfill，事实上，Babel作为前端工程化的基石，作用远不止这些  
比如，我在上篇文章{% post_link webpack/如何写一个babel插件 %}中介绍了babel plugin的一个实现，也介绍了babel的大致编译过程。实际上Babel生态中有很多概念，比如：preset、plugin、runtime等，这篇文章将会深入讲解babel  

#### Babel具体能力
Babel最常见的上层能力为：
1. polyfill
2. DSL转换（比如解析JSX）
3. 语法转换（比如将高级语法解析为当前可用的实现）
由于篇幅有限，这里仅介绍polyfill与「语法转换」相关功能。

#### polyfill
我们知道使用@babel/polyfill或@babel/preset-env可以实现高级语法的降级实现以及API的polyfill  
由于Babel本身只是JS的编译器，以上两者的转换功能是谁实现的呢？答案是core-js
core-js也是由多个库组成的，包括：
1. core-js-builder
2. core-js-bundle
3. core-js-compat
4. core-js-pure
5. core-js
##### core-js
```js
import 'core-js/features/array/from'; 
import 'core-js/features/array/flat'; 
import 'core-js/features/set';        
import 'core-js/features/promise';    

Array.from(new Set([1, 2, 3, 2, 1]));          // => [1, 2, 3]
[1, [2, 3], [4, [5]]].flat(2);                 // => [1, 2, 3, 4, 5]
Promise.resolve(32).then(x => console.log(x)); // => 32
```
但直接使用core-js会污染全局命名空间，可以用core-js-pure代替。

##### core-js-compat
core-js-compat根据Browserslist维护了不同宿主环境、不同版本下对应需要支持特性的集合

#### @babel/polyfill与core-js关系
@babel/polyfill可以看作是：core-js加regenerator-runtime
> regenerator-runtime是generator以及async/await的运行时依赖
> 注意：单独使用@babel/polyfill会将core-js全量导入，造成项目打包体积过大，从Babel v7.4.0开始，@babel/polyfill被废弃了，可以直接引用core-js与regenerator-runtime替代  

为了解决全量引入core-js造成打包体积过大的问题，我们需要配合使用@babel/preset-env   
上篇文章{% post_link webpack/如何写一个babel插件 %}介绍了babel插件，这里的@babel/preset-env就是多个插件的集合，它按照两个粒度来选择插件：
1. 宿主环境的粒度：根据不同宿主环境将该环境下所需的所有特性打包
2. 按使用情况的粒度：仅仅将使用了的特性打包

按照宿主环境的粒度来选择：当我们配置browserslist文件（或在@babel/preset-env的targets属性内设置，或在package.json的browserslist属性中设置）时，babel会自动引入配置环境需要的插件

按使用情况的粒度：设置@babel/preset-env的useBuiltIns属性为usage，就会只打包我们使用过的特性，相信这是webpack配置都了解的

#### @babel/runtime
在这个[babel playground](https://babeljs.io/repl#?browsers=&build=&builtIns=false&corejs=3.6&spec=false&loose=false&code_lz=MYGwhgzhAECCAO9oG8C-Q&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=script&lineWrap=true&presets=env&prettier=false&targets=&version=7.13.7&externalPlugins=babel-plugin-transform-regenerator%406.26.0&assumptions=%7B%7D)例子中，我们使用了class，可以看到编译出的结果如下：
```js
function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

var App = function App() {
  "use strict";

  _classCallCheck(this, App);
};
```
可以发现用到了_classCallCheck辅助方法  
那么，按理来说，我们项目中每个打包的文件是不是都应该添加这种辅助方法呢？  
不是  
@babel/runtime包含了Babel所有辅助方法以及regenerator-runtime；和@babel/polyfill一样，为了按需引用辅助方法，我们还需要使用@babel/plugin-transform-runtime，将@babel/plugin-transform-runtime置为devDependence，因为他在编译时使用。将@babel/runtime置为dependence，因为他在运行时使用。

#### 总结
了解这些是webpack工程化的基础，也能让我们更深入地理解babel的运行原理，对项目性能调优具有很重要的意义。
