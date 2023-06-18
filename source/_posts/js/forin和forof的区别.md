---
title: forin和forof的区别
date: 2019/11/13 20:42:21
categories: js
tags: js
top: 1
---

for in遍历数组会遍历到数组原型上的属性和方法, 更适合遍历对象

forEach不支持break, continue, return等

使用for of可以成功遍历数组的值, 而不是索引, 不会遍历原型

for in 可以遍历到myObject的原型方法method,如果不想遍历原型方法和属性的话，可以在循环内部判断一下,hasOwnPropery方法可以判断某属性是否是该对象的实例属性