---
title: vue项目的移动端适配
date: 2020/11/04 17:13:31
tags: [vue, css, npm, 浏览器]
categories: [vue, h5]
top: 3
toc: true
---
### vue项目的移动端适配
上篇文章{% post_link h5/移动端适配 %}谈到了移动端适配的一些解决方案，这篇文章主要谈谈vue项目中可以怎样做到移动端适配。
公司项目使用的vue-cli的3.x版本，有关移动端的配置引起了我的注意，下面介绍一下目前我们移动端项目采用的适配方案

#### 配置
首先，我们应该用NPM来安装postcss-px2rem：
```
npm i postcss-plugin-px2rem  --save -dev
```
然后在postcss.config.js配置插件参数：
```js
module.exports = {
  plugins: {
    autoprefixer: {},
    'postcss-plugin-px2rem': {
      rootValue: 75, // 换算基数， 默认75  ，这样的话把根标签的字体规定为1rem为75px,这样就可以从设计稿上量出多少个px直接在代码中写多上px了，例如你设计稿为750的宽度标准，那么这里的值设置为75则可。
      // unitPrecision: 5, // 允许REM单位增长到的十进制数字。
      // propWhiteList: [], // 默认值是一个空数组，这意味着禁用白名单并启用所有属性。
      // propBlackList: [], //黑名单
      exclude: /(node_module)/, // 默认false，可以（reg）利用正则表达式排除某些文件夹的方法，例如/(node_module)/ 。如果想把前端UI框架内的px也转换成rem，请把此属性设为默认值
      // selectorBlackList: [], //要忽略并保留为px的选择器
      // replace: true, // （布尔值）替换包含REM的规则，而不是添加回退。
      mediaQuery: false, // （布尔值）允许在媒体查询中转换px。
      minPixelValue: 3, // 设置要替换的最小像素值(3px会被转rem)。 默认 0
    },
  },
};

```
由于rem是根据根字体的大小来作为基准值的，然而我们的移动设备屏幕大小以及有些屏幕为视网膜屏的，会是普通屏幕的2倍，所以这个基准值我们需要根据不同设备来进行计算，我们在utils/rem.js里面添加如下代码，并在main.js里面引用：
```js
/* eslint-disable */
(function (doc, win) {
  var rem;
  var dpr = win.devicePixelRatio || 1;
  var docEl = doc.documentElement;
  var initialDeviceWidth = docEl.clientWidth;

  function setRemUnit() {
    var currentDeviceWith = docEl.clientWidth;
    var expectDetectionSize = currentDeviceWith / 10;
    var newRem;
    if (initialDeviceWidth === currentDeviceWith && win.navigator.userAgent.match(/Android/)) {
      // 解决安卓缩放字号引起的页面错乱 bug
      var renderDetectionSizeStr = window.getComputedStyle(docEl).fontSize;
      var renderDetectionSize = renderDetectionSizeStr.substr(0, renderDetectionSizeStr.length - 2);
      if (Math.abs(expectDetectionSize - renderDetectionSize) > 2) {
        var currentRem = parseFloat(docEl.style.fontSize) || expectDetectionSize;
        newRem = currentRem * (expectDetectionSize / renderDetectionSize);
        newRem = newRem.toFixed(1);
      }
    } else {
      newRem = expectDetectionSize;
      initialDeviceWidth = currentDeviceWith;
    }

    if (newRem && parseFloat(docEl.style.fontSize) !== newRem) {
      rem = newRem;
      docEl.style.fontSize = rem + 'px';
      win.rem = rem;
    }

    if (requestAnimationFrame) {
      requestAnimationFrame(setRemUnit);
    }
  }

  setRemUnit();

  win.dpr = dpr;
})(document, window);
```
上面的代码给页面[动态设置了fontSize](https://zhuanlan.zhihu.com/p/80692165)，并解决了[安卓手机设置个性字体和字体大小导致的页面混乱问题](https://segmentfault.com/q/1010000009571528)（需不需要解决得看具体需求），在浏览器中改变手机型号，会根据不同手机的dpr动态改变页面的font-size：
![pixel](http://liangdo-top.oss-cn-shenzhen.aliyuncs.com/blog/pixel.png) 
![iponex](http://liangdo-top.oss-cn-shenzhen.aliyuncs.com/blog/iphonex.png)

#### 总结
以上就是目前用到的一些移动端H5的解决方案，对于更复杂的场景，需要更灵活考虑，目前没有银弹。
