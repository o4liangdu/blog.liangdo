---
title: vue2组件初始化工作
date: 2021/04/02 17:09:12
tags: [vue, js]
categories: vue
top: 3
toc: true
---
### vue2中针对各个属性做了哪些工作
vue2在初始化一个vue实例的时候，会调用initState方法：
```js
function initState(vm) {
    vm._watchers = [];
    var opts = vm.$options;
    if(opts.props) {
        initProps(vm, opts.props); //初始化props
    }
    if(opts.methods) {
        initMethods(vm, opts.methods); //初始化methods
    }
    if(opts.data) {
        initData(vm); //初始化data
    } else {
        observe(vm._data = {}, true /* asRootData */ );
    }
    if(opts.computed) {
        initComputed(vm, opts.computed); //初始化computed
    }
    if(opts.watch && opts.watch !== nativeWatch) {
        initWatch(vm, opts.watch); //初始化watch
    }
}
```
可以看到，initState里面主要是对vue实例中的 props, methods, data, computed 和 watch 数据进行初始化，下面将对各个初始化工作进行分析

#### initProps
遍历props中的每个属性，然后进行类型验证，数据监测等（提供为props属性赋值就抛出警告的钩子函数）

#### initMethods
监测methods中的方法名是否合法

#### initData
运行 observe 函数深度遍历数据中的每一个属性，进行数据劫持

#### initComputed
监测数据是否已经存在data或props上，如果存在则抛出警告，否则调用defineComputed函数，监听数据，为组件中的属性绑定getter及setter。如果computed中属性的值是一个函数，则默认为属性的getter函数。此外属性的值还可以是一个对象，他只有三个有效字段set、get和cache，分别表示属性的setter、getter和是否启用缓存，其中get是必须的，cache默认为true

#### initWatch
调用vm.$watch函数为watch中的属性绑定setter回调（如果组件中没有该属性则不能成功监听，属性必须存在于props、data或computed中）。如果watch中属性的值是一个函数，则默认为属性的setter回调函数，如果属性的值是一个数组，则遍历数组中的内容，分别为属性绑定回调，此外属性的值还可以是一个对象，此时，对象中的handler字段代表setter回调函数，immediate代表是否立即先去执行里面的handler方法，deep代表是否深度监听。
>vm.$watch函数会直接使用Watcher构建观察者对象。watch中属性的值作为watcher.cb存在，在观察者update的时候，在watcher.run函数中执行  

> 另外，computed和watch都是基于Watcher实现的：[计算属性 vs 侦听属性
](https://cn.vuejs.org/v2/guide/computed.html#%E8%AE%A1%E7%AE%97%E5%B1%9E%E6%80%A7-vs-%E4%BE%A6%E5%90%AC%E5%B1%9E%E6%80%A7)