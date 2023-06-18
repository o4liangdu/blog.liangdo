---
title: 基于彩云天气api应用
date: 2019/5/13 20:56:25
categories: 实战项目
top: 2
tags: ["小程序", "js"]
toc: true
---
## 彩云天气api应用

用了很多天气应用，个人觉得播报比较准确的算是微信小程序的小伞天气了，但近来这个小程序恰饭的广告比较多，所以考虑自己做一个。

### 流程
1.抓包确定小伞天气采用的服务平台接口  
2.申请平台token  
3.技术选型，小程序搭建

#### 1.抓包确定小伞天气采用的服务平台接口
这是小伞天气的界面
![](http://liangdo-top.oss-cn-shenzhen.aliyuncs.com/blog/1.jpg)

> 用了这么久，有用的功能就未来两小时和明天是否下雨有用，其余都是虚的，那些云图什么的并没有什么参考价值，而且调用云图服务还要付费，所以这个需求砍掉  

通过[whistle](http://wproxy.org/whistle/)抓取小程序的包，发现这个小程序其实是调用的彩云api来获取天气降雨信息的

#### 申请token
到[彩云开放平台](https://dashboard.caiyunapp.com/v1/token/)申请到token（实测天气接口还挺容易通过的）  
阅读官方的api文档,选择调用其通用接口（包含了分钟、小时和天时间范围的天气信息）：https://open.caiyunapp.com/%E9%80%9A%E7%94%A8%E9%A2%84%E6%8A%A5%E6%8E%A5%E5%8F%A3/v2.5  
> 试着调用了下深圳的接口，好家伙，json包有近70k  

阅读文档后真正有用的字段是这些：
```
最近降水距离:$.result.realtime.precipitation.nearest.distance
本地降水强度:$.result.realtime.precipitation.local.intensity
本地未来1小时降水概率:$.result.minutely.probability
实时自然语言描述:$.result.minutely.description
逐小时预报自然语言描述:$.result.hourly.description

```
可以选择加入下面这些信息的字段：
```
温度：$.result.realtime.temperature
风速：$.result.realtime.wind.speed
紫外线指数:$.result.realtime.life_index.ultraviolet.index
PM25浓度:$.result.realtime.air_quality.pm25
```


#### 小程序构建
由于自己有做一个小程序，就是整合一些小功能的，确实方便了我不少，替代了好几个臃肿的app，所以决定把这个功能整合进去。
这个功能流程如下：
1.调用微信小程序获取位置权限获得当前经纬度
2.利用经纬度数据调用彩云api获取天气数据
回显到小程序效果如下：
![](https://liangdo-top.oss-cn-shenzhen.aliyuncs.com/blog/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202021-08-06%20%E4%B8%8A%E5%8D%8812.51.12.png)

> 后续会增加城市搜索的功能,并升级为线上版本。tobe continue
