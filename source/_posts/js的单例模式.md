---
title: js的单例模式
date: 2020-02-01 09:27:16
tags: [js, 设计模式]
categories: js
top: 3
toc: true
---
#### 单例模式特点
顾名思义，单例模式就是保证一个类仅有一个实例，并提供一个访问它的全局访问点。  
单例模式在在各个层次的应用中随处可见，比如系统层面windows中的资源管理器、web中点击登录弹框唯一性、vuex中store对象等等，这些场景都用到了单例模式  
单例模式的具体特点如下：
+ 单例类只有一个实例；
+ 单例类在该类的内部创建自身的实例对象
+ 向整个系统公开这个实例接口

优点如下：可以减少不必要的内存开销，并且在减少全局的函数和变量冲突也有不可替代的作用

#### 从全局变量出发认识单例模式
下面代码用到了一个全局对象，每次要用到他的时候就检查一下是否存在，存在就拿过来用，没有就初始化一个新的：
```js
let uniqueObj = null;
// 初始化函数
function init() {
    uniqueObj = {
        name: "11"
    }
}
if(uniqueObj) {
    // use it
} else {
    init()
}
// ...
```
为了利于更好的扩展性，我们通常将这个“全局变量”封装成一个单例，这样，不管在哪调用，都是唯一确定的，我们可以这样构建一个单例：
```js
var Singleton = function (name) {
  this.name = name;
};

Singleton.getInstance = (function () {
  var instance = null;

  return function (name) {
    if (!instance) {
      instance = new Singleton(name);
    }
    return instance;
  };
})();
const a = Singleton.getInstance("11")
const b = Singleton.getInstance("22")
// 这里a和b指向的同一个实例
console.log(a===b, a)
```

