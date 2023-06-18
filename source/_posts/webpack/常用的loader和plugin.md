---
title: webpack常用的loader和plugin
date: 2020/07/17 12:14:36
tags: [webpack, nodejs, js]
categories: webpack
top: 3
toc: true
---
### 常用的loader和plugin
这里总结一下webpack中常用的loader和plugin

#### 常用loader
##### css相关：
css-loader：处理 import / require（） @import / url 引入的内容  
style-loader：将css-loader解析后的内容挂载到html页面当中  
less-loader和sass-loader： 将less、sass预处理语法转换成css
```
npm install --save-dev style-loader css-loader less-loader sass-loader
```
##### 字体图片相关：
raw-loader：将文件作为字符串导入  
file-loader:处理一系列的图片文件；比如：.png 、 .jpg 、.jepg等格式的图片,并给每张图片都生成一个随机的hash值作为图片的名字  
url-loader：处理一系列的图片文件，可以将设置一定大小内的图片转为base64编码，内联到html中去
vue-loader： 处理vue文件
```
npm install --save-dev file-loader raw-loader url-loader
```
##### eslint相关：
eslint-loader:审查代码是否符合编码规范和统一的代码风格；审查代码是否存在语法错误
```
npm install --save-dev eslint-loader
```
##### 编译相关
babel-loader: babel实现如下功能：
1. polyfill
2. DSL转换（比如解析JSX）
3. 语法转换（比如将高级语法解析为当前可用的实现）  

ts-loader: ts->js
```
npm install --save-dev babel-loader ts-loade
```

#### 常用plugin
插件比较多，功能也很丰富，就没有专门分类了，这里罗列整理一些：
1. ProvidePlugin：自动加载模块，代替require和import
2. html-webpack-plugin可以根据模板自动生成html代码，并自动引用css和js文件
3. extract-text-webpack-plugin 将js文件中引用的样式单独抽离成css文件
4. DefinePlugin 编译时配置全局变量，这对开发模式和发布模式的构建允许不同的行为非常有用。
5. HotModuleReplacementPlugin 热更新
6. optimize-css-assets-webpack-plugin 不同组件中重复的css可以快速去重
7. webpack-bundle-analyzer 一个webpack的bundle文件分析工具，本地起一个服务，将bundle文件以可交互缩放的treemap的形式展示。
8. compression-webpack-plugin 生产环境可采用gzip压缩JS和CSS
9. happypack：通过多进程模型，来加速代码构建
10. clean-wenpack-plugin 清理每次打包下没有使用的文件
11. script-ext-html-webpack-plugin 将比较小的runtime包内联
12. extract-text-webpack-plugin 单独打包css