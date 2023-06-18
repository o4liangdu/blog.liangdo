---
title: 'Vary: Origin解决条件CORS所可能引发的错乱问题 - Do丶'
tags: ["浏览器", "js"]
categories: debug
excerpt: "CORS，全名为跨域资源共享，是为了让不同网站的页面之间互相访问数据的机制。简单来说，CORS 的工作机制是这样的：网站 A 请求网站 B 的资源，网站 A 发起的请求会在\_Origin\_请求头上带上自己的源（origin）信息，如果网站 B 返回的响应头里有Access-Control-Allow"
date: 2020-08-29 15:08:00
---

CORS，全名为跨域资源共享，是为了让不同网站的页面之间互相访问数据的机制。简单来说，CORS 的工作机制是这样的：网站 A 请求网站 B 的资源，网站 A 发起的请求会在 Origin 请求头上带上自己的源（origin）信息，如果网站 B 返回的响应头里有Access-Control-Allow
<!-- more -->
【摘要】 [阅读全文](http://www.cnblogs.com/Leo-Do/p/13582308.html)