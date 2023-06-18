---
title: benchmarkjs 测试函数性能
date: 2020-05-14 10:31:21
tags: [js, 测试]
categories: 前端性能优化
top: 2
toc: true
---
在对性能要求比较高的一些项目中，通常需要进行性能测试，这里介绍一个我用过的性能测试工具[benchmarkjs](https://www.npmjs.com/package/benchmark)

#### 安装
```js
npm i --save benchmark
```
> 在node环境下测试性能，可以安装[microtime](https://www.npmjs.com/package/microtime)库，一个用c++写的获取高精度时间戳的包  

#### 使用
```js
var Benchmark = require("benchmark")
var suite = new Benchmark.Suite;
 
// add tests
suite.add('RegExp#test', function() {
  /o/.test('Hello World!');
})
.add('String#indexOf', function() {
  'Hello World!'.indexOf('o') > -1;
})
// add listeners
.on('cycle', function(event) {
  // 这里打印函数多次执行耗时
  console.log(String(event.target));
})
.on('complete', function() {
  // 这里打印结果
  console.log('Fastest is ' + this.filter('fastest').map('name'));
})
// run async
.run({ 'async': true });
```
以上代码添加了两个命名的函数并配置cycle循环执行，打印出耗时，比较出结果，下面是我的运行结果：
```js
RegExp#test x 26,935,718 ops/sec ±3.63% (79 runs sampled)
String#indexOf x 734,602,605 ops/sec ±1.06% (90 runs sampled)
Fastest is String#indexOf
```
通过这个包，我们可以快速比较两种方法的执行效果，在平时的代码优化测试中还是很常用的。
