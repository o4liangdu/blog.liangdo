---
title: underscore源码学习
date: 2021/05/11 12:30:30
tags: [源码]
categories: js
top: 3
toc: true
---
最近学习了underscore的部分源码，在此总结下  
#### 基于IIFE的模块化
underscore最外层是一个立即执行函数，回顾一下立即执行函数(IIFE)的主要特征：  
1. 使用函数表达式声明一个函数
2. 在其后使用括号直接调用
使用立即函数可以创建一个独立的沙箱似的作用域，它是函数的一种特殊调用方式，利用了JavaScript的函数作用域的概念。这样可以防止其他代码对该函数内部造成影响，而且不会创造全局变量防止污染全局空间

#### 确认环境的全局命名空间
基于能力检测，将全局命名空间赋值给root：
```js
var
  root =
  (
    (
      (
        ((typeof(self)) == ("object")) // self表示window窗口自身，这是浏览器环境下的全局命名空间
        &&
        ((self.self) === (self)) // 如果存在self，判断self是否是自身引用，即window这一对象
      )
      &&
      (self) // 如果以上都满足，说明全局对象是window，并返回window作为root，这里self即window
    )
    ||
    (
      (
        ((typeof(global)) == ("object")) // global表示node环境下全局的命名空间
        &&
        ((global.global) === (global)) // 如果存在gloabl，判断global是否是自身引用
      )
      &&
      (global) // 如果以上都满足，说明全局对象是global，并返回global作为root
    )
  )
  ||
  (this); // 如果以上都不满足，直接返回this，这里应该处理既不是window这一浏览器环境，也不是global这一node环境的
```

#### 防止冲突
underscore基于全局变量"_"，为了防止冲突，做了如下处理：
```js
// 保存原有全局变量“_”,root是全局命名空间
var previousUnderscore = root._;

// 使用noConflict方法返回自身
_.noConflict = function() {
  root._ = previousUnderscore;
  return this;
};
```

#### Object.create的兼容
首先回顾下Object.create的作用：Object.create(proto, [propertiesObject])，方法创建一个新对象，使用现有的对象来提供新创建的对象的proto。怎么理解这句话呢，我是这样理解的：第一个参数是放在新对象的原型上的，第二个参数是放在新对象的实例上的。  
在underscore中做了如下处理：
```js
// 方法快速引用
var nativeCreate = Object.create;
// 用于代理原型转换的空函数
var Ctor = function() {};  
var baseCreate = function(prototype) {
  if (!(_.isObject(prototype))) return {}; // 如果参数不是对象，直接返回空对象
  if (nativeCreate) return nativeCreate(prototype); // 如果原生的对象创建可以使用，返回该方法根据原型创建的对象
  // 处理没有原生对象创建的情况
  Ctor.prototype = prototype;  // 将空函数的原型指向要使用的原型
  var result = new Ctor();  // 创建一个实例
  Ctor.prototype = null;  // 恢复Ctor的原型供下次使用
  return result;  // 返回该实例
};
```

