---
title: web中常见的内存泄漏
date: 2021-08-05 10:21:11
tags: [js, 浏览器, 内存泄漏]
categories: 前端性能优化
top: 3
toc: true
---
## web中的内存泄漏
### 什么是内存泄漏
js引擎中有垃圾回收机制，它主要针对一些程序中不再使用的对象，对其清理回收释放掉内存。虽然针对垃圾回收做了各种优化从而尽可能的确保垃圾得以回收，但并不是说我们就可以完全不用关心这块了，我们代码中依然要主动避免一些不利于引擎做垃圾回收操作，因为不是所有无用对象内存都可以被回收的，那当不再用到的对象内存，没有及时被回收时，我们叫它内存泄漏（Memory leak）
### 常见的内存泄漏
#### 不正当的闭包
> js闭包和内存泄漏的关系
``` js
function fn1(){
  let test = new Array(1000).fill('isboyjc')
  return function(){
    console.log('hahaha')
  }
}
let fn1Child = fn1()
fn1Child()
```
上面这个代码是一个闭包，但因为返回的函数中并没有对 fn1 函数内部的引用，test 变量完全是可以被回收的，所以并没有造成内存泄漏。
``` js
function fn2(){
  let test = new Array(1000).fill('isboyjc')
  return function(){
    console.log(test)
    return test
  }
}
let fn2Child = fn2()
fn2Child()
```
与之对应的上面这段代码则造成了内存泄漏。
> 如何解决？  

在函数调用后，把外部的引用关系置空就好了，如下：
```js
function fn2(){
  let test = new Array(1000).fill('isboyjc')
  return function(){
    console.log(test)
    return test
  }
}
let fn2Child = fn2()
fn2Child()
fn2Child = null
```

#### 隐式全局变量
函数中的局部变量在函数执行结束后这些变量已经不再被需要，所以垃圾回收器会识别并释放它们。但是对于全局变量，垃圾回收器很难判断这些变量什么时候才不被需要，所以全局变量通常不会被回收:
```js
function fn(){
  // 没有声明从而制造了隐式全局变量test1
  test1 = new Array(1000).fill('isboyjc1')
  
  // 函数内部this指向window，制造了隐式全局变量test2
  this.test2 = new Array(1000).fill('isboyjc2')
}
fn()
```
> 解决方法也比较简单，在使用完全局变量将其置为null即可
#### 游离DOM引用
代码中进行 DOM 时会使用变量缓存 DOM 节点的引用，但移除节点的时候，我们应该同步释放缓存的引用，否则游离的子树无法释放：
```html
<div id="root">
  <ul id="ul">
    <li></li>
    <li></li>
    <li id="li3"></li>
    <li></li>
  </ul>
</div>
<script>
  let root = document.querySelector('#root')
  let ul = document.querySelector('#ul')
  let li3 = document.querySelector('#li3')
  
  // 由于ul变量存在，整个ul及其子元素都不能GC
  root.removeChild(ul)
  
  // 虽置空了ul变量，但由于li3变量引用ul的子节点，所以ul元素依然不能被GC
  ul = null
  
  // 已无变量引用，此时可以GC
  li3 = null
</script>
```
#### 遗忘的定时器
```js
// 获取数据
let someResource = getData()
setInterval(() => {
  const node = document.getElementById('Node')
 if(node) {
    node.innerHTML = JSON.stringify(someResource))
 }
}, 1000)
```
在 setInterval 没有被clearInterval结束前，回调函数里的变量以及回调函数本身都无法被回收
> 另外，浏览器中的 requestAnimationFrame 也存在这个问题，我们需要在不需要的时候用 cancelAnimationFrame API 来取消使用
#### 遗忘的事件监听器
当事件监听器在组件内挂载相关的事件处理函数，而在组件销毁时不主动将其清除时，其中引用的变量或者函数都被认为是需要的而不会进行回收，如果内部引用的变量存储了大量数据，可能会引起页面占用内存过高，这样就造成意外的内存泄漏。  
这里以vue使用情景为例：
```js
<template>
  <div></div>
</template>

<script>
export default {
  created() {
    window.addEventListener("resize", this.doSomething)
  },
  beforeDestroy(){
    window.removeEventListener("resize", this.doSomething)
  },
  methods: {
    doSomething() {
      // do something
    }
  }
}
</script>
```
#### 遗忘的监听者模式
对于目前的前端开发框架来说，监听者模式实现一些消息通信都是非常常见的，比如 EventBus:
```js
<template>
  <div></div>
</template>

<script>
export default {
  created() {
    eventBus.on("test", this.doSomething)
  },
  beforeDestroy(){
    eventBus.off("test", this.doSomething)
  },
  methods: {
    doSomething() {
      // do something
    }
  }
}
</script>
```
#### 遗忘的Map、Set对象
当使用 Map 或 Set 存储对象时，同 Object 一致都是强引用，如果不将其主动清除引用，其同样会造成内存不自动进行回收。

如果使用 Map ，对于键为对象的情况，可以采用 WeakMap，WeakMap 对象同样用来保存键值对，对于键是弱引用（注：WeakMap 只对于键是弱引用），且必须为一个对象，而值可以是任意的对象或者原始值，由于是对于对象的弱引用，不会干扰 Js 的垃圾回收。

如果需要使用 Set 引用对象，可以采用 WeakSet，WeakSet 对象允许存储对象弱引用的唯一值，WeakSet 对象中的值同样不会重复，且只能保存对象的弱引用，同样由于是对于对象的弱引用，不会干扰 Js 的垃圾回收。

下面是🌰 ：
```js
let obj = {id: 1}
let user = {info: obj}
let set = new Set([obj])
let map = new Map([[obj, 'hahaha']])

// 重写obj
obj = null 

console.log(user.info) // {id: 1}
console.log(set)
console.log(map)
```
上面代码我们重写 obj 以后，{id: 1} 依然会存在于内存中，因为 user 对象以及后面的 set/map 都强引用了它，Set/Map、对象、数组对象等都是强引用，所以我们仍然可以获取到 {id: 1} ，我们想要清除那就只能重写所有引用将其置空了。  
接下来我们看 WeakMap 以及 WeakSet：
```js
let obj = {id: 1}
let weakSet = new WeakSet([obj])
let weakMap = new WeakMap([[obj, 'hahaha']])

// 重写obj引用
obj = null 
// {id: 1} 将在下一次 GC 中从内存中删除
```
#### 未清理的Console输出
console 如果输出了对象也会造成内存泄漏，大家可以试下如下例子
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>test</title>
</head>

<body>
  <button id="click">click</button>

  <script>
    !function () {
      function Test() {
        this.init()
      }
      Test.prototype.init = function () {
        this.a = new Array(10000).fill('isboyjc')
        console.log(this)
      }

      document.querySelector('#click').onclick = function () {
        new Test();
      }
    }()
  </script>
</body>

</html>
```

关于web常见内存泄漏场景就说道这了，下回更新内存泄漏排查、定位与修复。
tobe continue
