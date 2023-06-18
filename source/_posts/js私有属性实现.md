---
title: js私有属性
date: 2020-07-17 12:22:11
tags: [js, 设计模式]
categories: js
top: 3
toc: true
---
### js私有属性
JavaScript被很多人认为并不是一种面向对象语言，原因有很多种，比如JavaScript没有类，不能提供传统的类式继承；再比如JavaScript不能实现信息的隐藏，不能实现私有成员。但还是有很多方法可以"间接实现"js的私有属性。

#### js基于编码规范约定实现方式
```js
function Person(name){
  this._name = name;
}

var person = new Person('11');
```
可以发现以上的实现方式只是一个约定_为私有属性而已，我们还是可以通过_name访问到实例的属性。

#### 基于闭包的实现方式
```js
function Person(name){
  var _name = name;
  this.getName = function(){
    return _name;
  }
}

var person = new Person('11');
```
基于闭包的实现被广泛地使用，但也有人提出了这种实现方式的一些缺陷：
+ 私有变量和特权函数只能在构造函数中创建。通常来讲，构造函数的功能只负责创建新对象，方法应该共享于prototype上。特权函数本质上是存在于每个实例中的，而不是prototype上，增加了资源占用。  
上面这句话不好懂，大意是说上面的getName应该是存在于Person.prototype中被实例共用的，而不是作为一个实例的方法，这样的作法会增加实例的资源占用，不好不好。  

#### 基于强引用散列表的实现方式
es5不支持Map数据结构，可以给每个实例新增一个唯一的标识符，以此标识符为key，对应的value便是这个实例的私有属性，这对key-value保存在一个Object内：
```js
var Person = (function() {
    var privateData = {},
        privateId = 0;

    function Person(name) {
        Object.defineProperty(this, "_id", { value: privateId++ });

        privateData[this._id] = {
            name: name
        };
    }
    Person.prototype.getName = function() {
        return privateData[this._id].name;
    };

    return Person;
}());
```
现在把getName放到Person.prototype中被实例共用了，是不是就好了呢？并不，因为privateData存储了每个实例的私有属性name的key-value，对每个实例都是强引用，导致实例是不能被垃圾回收处理的，如果实例多了就会导致内存泄漏
> 关于js常见的内存泄漏可以参考我的这篇文章：

#### 基于WeakMap的实现方式
在es6中，有一个WeakMap的数据类型，它有如下特点：
1.支持使用对象类型作为key值  
2.弱引用  
基于WeakMap的特点，便不必为每个实例都创建一个唯一标识符，因为实例本身便可以作为WeakMap的key。改进后的代码如下：
```js
var Person = (function() {

    var privateData = new WeakMap();

    function Person(name) {
        privateData.set(this, { name: name });
    }

    Person.prototype.getName = function() {
        return privateData.get(this).name;
    };

    return Person;
}());
```
WeakMap是一种弱引用散列表， 这意味着,如果没有其他引用和该键引用同一个对象,这个对象将会被当作垃圾回收掉。解决了内存泄露的问题。
这个方法的缺点是浏览器的WeakMap兼容性不太理想，投入生产仍需慎重考虑。
