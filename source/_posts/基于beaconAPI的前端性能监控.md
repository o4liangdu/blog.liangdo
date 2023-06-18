---
title: 基于beacon API和Navigation Timing API的前端性能监控
date: 2021/5/08 16:54:21
tags: [js, 浏览器, 性能监控]
categories: 前端性能优化
top: 3
toc: true
---
#### 应用场景

为了解决网页卸载时，异步请求无法成功的问题，浏览器特别实现了一个 Beacon API，允许异步请求脱离当前主线程，放到浏览器进程里面发出，这样可以保证一定能发出。
因此Beacon API 不会延缓网页卸载，不会严重影响用户体验。
相应的主要应用场景则是数据上报、异常处理和性能监控方面

#### 浏览器支持情况

![](http://liangdo-top.oss-cn-shenzhen.aliyuncs.com/blog/beacon.png)
可以通过下面js代码判断浏览器是否支持beaconAPI,如果不支持则可以降级到XHR或者使用的img标签get请求：

```js
if (navigator.sendBeacon) {
    // Beacon 代码
} else {
    // 不支持 Beacon。可以降级到 XHR
    // 这里用img的get做降级，没有兼容性问题，可以将参数拼接到url
    var i = new Image();
    i.onload = i.onerror = i.onabort = function () {
    i = i.onload = i.onerror = i.onabort = null;
    }
    i.src = url;
}
```

#### 接口使用

navigator.sendBeacon(url, data),其中，第一个参数是发送请求的 URL。该请求作为 HTTP POST 请求执行，发送在第二个参数中提供的任意数据。  
data的格式可以是：Blob，BufferSource，FormData，或者 URLSearchParams - 基本上包括任一使用 Fetch 发送请求时的 body 类型，这个参数是可选的。  
sendBeacon是有返回值的，类型为bool：true表示浏览器已经将这个请求纳入队列稍后处理，false表示浏览器无法完成这个请求，其原因不详，不过通常来说就是浏览器的HTTP请求队列已满。
以FormData为例，构造如下基于BeaconAPI的请求：

```js
let url = '/api/example';

// 创建一个 FromData 并新增一个键值对
let data = new FormData();
data.append('hello', 'world');

let result = navigator.sendBeacon(url, data);

if (result) { 
  console.log('Successfully queued!');
} else {
  console.log('Failure.');
}
```

> 如果用户浏览器支持DNT（DO NOT TRACK），会在http头添加一个header:  
DNT: 1
那么我们最好遵照该用户的意愿，把该记录匿名化

#### 实践应用

配合[Navigation Timing API](https://developer.mozilla.org/zh-CN/docs/Web/API/Navigation_timing_API)，可以实现精准的页面性能数据上报,这里附上一张浏览器加载页面流程图：
![](http://liangdo-top.oss-cn-shenzhen.aliyuncs.com/blog/3600712-117295fb088c7637.webp)
以下是各个时间节点的描述：

```js
navigationStart: 加载起始时间
redirectStart: 重定向开始时间（如果发生了HTTP重定向，每次重定向都和当前文档同域的话，就返回开始重定向的fetchStart的值。其他情况，则返回0）
redirectEnd: 重定向结束时间（如果发生了HTTP重定向，每次重定向都和当前文档同域的话，就返回最后一次重定向接受完数据的时间。其他情况则返回0）
fetchStart: 浏览器发起资源请求时，如果有缓存，则返回读取缓存的开始时间
domainLookupStart: 查询DNS的开始时间。如果请求没有发起DNS请求，如keep-alive，缓存等，则返回fetchStart
domainLookupEnd: 查询DNS的结束时间。如果没有发起DNS请求，同上
connectStart: 开始建立TCP请求的时间。如果请求是keep-alive，缓存等，则返回domainLookupEnd
secureConnectionStart: 如果在进行TLS或SSL，则返回握手时间
connectEnd: 完成TCP链接的时间。如果是keep-alive，缓存等，同connectStart
requestStart: 发起请求的时间
responseStart: 服务器开始响应的时间
domLoading: 从图中看是开始渲染dom的时间，具体未知
domInteractive: 未知
domContentLoadedEventStart: 开始触发DomContentLoadedEvent事件的时间
domContentLoadedEventEnd: DomContentLoadedEvent事件结束的时间
domComplete: 从图中看是dom渲染完成时间，具体未知
loadEventStart: 触发load的时间，如没有则返回0
loadEventEnd: load事件执行完的时间，如没有则返回0
unloadEventStart: unload事件触发的时间
unloadEventEnd: unload事件执行完的时间
```

与网页性能的几个关键时间可以计算得到：

+ DNS解析时间： domainLookupEnd - domainLookupStart
+ TCP建立连接时间： connectEnd - connectStart
+ 白屏时间： responseStart - navigationStart
+ dom渲染完成时间： domContentLoadedEventEnd - navigationStart
+ 页面onload时间： loadEventEnd - navigationStart

> ps: Navigation Timing兼容性比BeaconAPI要好上不少，实际应用中主要考虑Beacon的支持
通过window.performance.timing所获的的页面渲染所相关的数据，在SPA应用中改变了url但不刷新页面的情况下是不会更新的。因此仅仅通过该api是无法获得每一个子路由所对应的页面渲染的时间。如果需要上报切换路由情况下每一个子页面重新render的时间，需要另外采用时间统计api

#### 总结

结合BeaconAPI和相应降级方案，可以实现前端监控接口的发送  
Navigation Timing API可以监控大部分前端页面的性能。但随着SPA模式的盛行，类似vue，reactjs等框架的普及，页面内容渲染的时机被改变了，需要我们灵活运用  
目前W3C关于首屏统计已经进入了提议阶段，各浏览器厂商正在打造更能代表用户使用体验的FP、FCP、FMP指标，将来会逐步开放API  
