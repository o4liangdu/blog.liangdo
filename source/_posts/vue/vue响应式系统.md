---
title: vue响应式系统
date: 2021/04/08 17:09:11
tags: [vue, js]
categories: vue
top: 3
toc: true
---
### 
在上篇文章{% post_link vue/发布-订阅模式 %}中，我们介绍了发布订阅模式的实现，这篇文章将基于上篇文章，详细解剖vue响应式系统的一些实现细节。  
vue 的响应式系统基于发布订阅模式，构建了三个重要的类：Dep 类、Watcher 类、Observer 类  

#### Observer
Observe扮演的角色是发布者，他的主要作用是**调用defineReactive函数**，在defineReactive函数中使用Object.defineProperty 方法对对象的每一个子属性进行数据劫持/监听  
其中，defineReactive函数是Observe的核心,功能是劫持数据，在**getter中向Dep（调度中心）添加观察者，在setter中通知观察者更新**  
以下是部分源码及注释：
```js
function defineReactive(obj, key, val, customSetter, shallow){
    //监听属性key
    //关键点：在闭包中声明一个Dep实例，用于保存watcher实例
    var dep = new Dep();

    var getter = property && property.get;
    var setter = property && property.set;
    
    if(!getter && arguments.length === 2) {
        val = obj[key];
    }
    //执行observe，监听属性key所代表的值val的子属性
    var childOb = observe(val);
    Object.defineProperty(obj, key, {
        enumerable: true,
        configurable: true,
        get: function reactiveGetter() {
            //获取值
            var value = getter ? getter.call(obj) : val;
            //依赖收集：如果当前有活动的Dep.target(观察者--watcher实例)
            if(Dep.target) {
                //将dep放进当前观察者的deps中，同时，将该观察者放入dep中，等待变更通知
                dep.depend();
                if(childOb) {
                    //为子属性进行依赖收集
                    //其实就是将同一个watcher观察者实例放进了两个dep中
                    //一个是正在本身闭包中的dep，另一个是子属性的dep
                    childOb.dep.depend();
                }
            }
            return value
        },
        set: function reactiveSetter(newVal) {
            //获取value
            var value = getter ? getter.call(obj) : val;
            if(newVal === value || (newVal !== newVal && value !== value)) {
                return
            }
            if(setter) {
                setter.call(obj, newVal);
            } else {
                val = newVal;
            }
            //新的值需要重新进行observe，保证数据响应式
            childOb = observe(newVal);
            //关键点：遍历dep.subs，通知所有的观察者
            dep.notify();
        }
    });
}
```
#### Dep
Dep 扮演的角色是**调度中心/订阅器**，主要的作用就是**收集观察者Watcher和通知观察者目标更新**。**每个属性拥有自己的消息订阅器dep**，用于存放所有订阅了该属性的观察者对象，当数据发生改变时，会遍历观察者列表（dep.subs），通知所有的watch，让订阅者执行自己的update逻辑  
以下是部分源码及注释：
```js
//Dep构造函数
var Dep = function Dep() {
    this.id = uid++;
    this.subs = [];
};
//向dep的观察者列表subs添加观察者
Dep.prototype.addSub = function addSub(sub) {
    this.subs.push(sub);
};
//从dep的观察者列表subs移除观察者
Dep.prototype.removeSub = function removeSub(sub) {
    remove(this.subs, sub);
};
Dep.prototype.depend = function depend() {
    //依赖收集：如果当前有观察者，将该dep放进当前观察者的deps中
    //同时，将当前观察者放入观察者列表subs中
    if(Dep.target) {
        Dep.target.addDep(this);
    }
};
Dep.prototype.notify = function notify() {
    // 循环处理，运行每个观察者的update接口
    var subs = this.subs.slice();
    for(var i = 0, l = subs.length; i < l; i++) {
        subs[i].update();
    }
};

//Dep.target是观察者，这是全局唯一的，因为在任何时候只有一个观察者被处理。
Dep.target = null;
//待处理的观察者队列
var targetStack = [];

function pushTarget(_target) {
    //如果当前有正在处理的观察者，将他压入待处理队列
    if(Dep.target) {
        targetStack.push(Dep.target);
    }
    //将Dep.target指向需要处理的观察者
    Dep.target = _target;
}

function popTarget() {
    //将Dep.target指向栈顶的观察者，并将他移除队列
    Dep.target = targetStack.pop();
}
```
#### Watcher
Watcher扮演的角色是订阅者/观察者，他的主要作用是为观察属性提供回调函数以及收集依赖（如计算属性computed，vue会把该属性所依赖数据的dep添加到自身的deps中），当被观察的值发生变化时，会接收到来自dep的通知，从而触发回调函数  
Watcher类的实现比较复杂，实例分为渲染 watcher（render-watcher）、计算属性 watcher（computed-watcher）、侦听器 watcher（normal-watcher）三种，
这三个实例分别是在三个函数中构建的：mountComponent 、initComputed和Vue.prototype.$watch。

**normal-watcher**：我们在组件钩子函数 **watch** 中定义的，都属于这种类型，即只要监听的属性改变了，都会触发定义好的回调函数，这类watch的expression是计算属性中的属性名。

**computed-watcher**：我们在组件钩子函数 **computed** 中定义的，都属于这种类型，每一个 computed 属性，最后都会生成一个对应的 watcher 对象，但是这类 watcher 有个特点：当计算属性依赖于其他数据时，属性并不会立即重新计算，只有之后其他地方需要读取属性的时候，它才会真正计算，即具备 lazy（懒计算）特性。这类watch的expression是我们写的回调函数的字符串形式。

**render-watcher**：每一个组件都会有一个 render-watcher, 当 data/computed 中的属性改变的时候，会调用该 render-watcher 来**更新组件的视图**。这类watch的expression是 function () {vm._update(vm._render(), hydrating);}。

除了功能上的区别，这三种 watcher 也有固定的执行顺序，分别是：computed-render -> normal-watcher -> render-watcher。  

#### 总结
Observe是对数据进行监听，Dep是一个订阅器，每一个被监听的数据都有一个Dep实例，Dep实例里面存放了N多个订阅者（观察者）对象watcher。

被监听的数据进行取值操作时（getter），如果存在Dep.target（某一个观察者），则说明这个观察者是依赖该数据的（如计算属性中，计算某一属性会用到其他已经被监听的数据，就说该属性依赖于其他属性，会对其他属性进行取值），就会把这个观察者添加到该数据的订阅器subs里面，留待后面数据变更时通知（会先通过观察者id判断订阅器中是否已经存在该观察者），同时该观察者也会把该数据的订阅器dep添加到自身deps中，方便其他地方使用。

被监听的数据进行赋值操作时（setter）时，就会触发dep.notify()，循环该数据订阅器中的观察者，进行更新操作。
