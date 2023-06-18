---
title: link中prefetch预加载功能
date: 2020/3/08 13:12:52
tags: [js, 浏览器]
categories: 前端性能优化
top: 3
toc: true
---
在H5中,有个很有用但常被忽略的特性,就是预先加载(prefetch),它的原理是利用浏览器的空闲时间去先下载用户指定需要的内容,然后缓存起来,这样用户下次加载时,就直接从缓存中取出来,效率就快了,这对弱网环境下的体验还是有很大提升的。

#### 什么情况下使用
对于一些使用场景强关联的页面，比如欢迎页到内容页，用户大概率会通过欢迎页进内容页，可以在欢迎页的空闲时间就预加载内容页或者影响内容页加载速度的图片等静态资源。

#### 怎么判断当前环境支不支持
对于为了实现优雅降级，我们需要判断浏览器是否支持预加载：
```js
const isPrefetchSupported = () => {
  const link = document.createElement('link');
  const { relList } = link;
 
  if (!relList || !relList.supports) {
    returnfalse;
  }
  return relList.supports('prefetch');
};
const prefetch = () => {
    const isPrefetchSupport = isPrefetchSupported();
    if (isPrefetchSupport) {
      const link = document.createElement('link');
      link.rel = 'prefetch';
      link.as = type;
      link.href = url;
      document.head.appendChild(link);
    } elseif (type === 'script') {
            // load script
    }
  };
```
以下是目前的支持情况，是否使用需要权衡：
![](http://liangdo-top.oss-cn-shenzhen.aliyuncs.com/blog/prefetch.png)

#### 类型和使用方法
+ 页面和图片资源可以像下面这样使用：
```html
<link rel="prefetch" href="http://www.example.com/">
<link rel="prefetch" href="/images/test.jpg"/>   
```

+ DNS预加载
获取域名对应的ip地址有时候会很慢，特别是在弱网环境和没配置好dns服务器的情况下，对性能要求高的场景，比如支付，影响还是很大的。可以通过下面方式实现：
```html
<meta http-equiv='x-dns-prefetch-control' content='on'>  
<link rel='dns-prefetch' href='http://g-ecx.images-amazon.com'>  
<link rel='dns-prefetch' href='http://z-ecx.images-amazon.com'>  
<link rel='dns-prefetch' href='http://ecx.images-amazon.com'>  
<link rel='dns-prefetch' href='http://completion.amazon.com'>  
<link rel='dns-prefetch' href='http://fls-na.amazon.com'>
```
> dns使用前提是资源已经部署好cdn了

+ css、js资源预加载
对于一些上报接口，如sentry等，如果为了用户体验考虑，可以延时加载，sentry官方也提供了一套完整的延时加载的方案；而对于一些非关键js文件，为了下一个页面尽快加载，可以牺牲当前页面的空闲时间，不过也得配合webpack，具体情况具体应用。

> 注意：只有可缓存的资源才进行预加载，否则浪费资源！
当资源为以下列表中的资源时，将阻止预渲染操作：  

1.URL 中包含下载资源  
2.页面中包含音频、视频  
3.POST、PUT 和 DELETE 操作的 ajax 请求  
4.HTTP 认证(Authentication)  
5.HTTPS 页面  
6.含恶意软件的页面  
7.弹窗页面  
8.占用资源很多的页面  
9.打开了 chrome developer tools 开发工具
