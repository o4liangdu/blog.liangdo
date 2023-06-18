---
title: "@font-face属性详解"
date: 2020/5/3 22:09:11
categories: css
tags: css
top: 2
---

## @font-face 属性详解

### 作用

@font-face 是 CSS3 中的一个模块，他主要是把自己定义的 Web 字体嵌入到你的网页中。

### 语法

```
@font-face {
    font-family: <YourWebFontName>;
    src: <source> [<format>][,<source> [<format>]]*;
    [font-weight: <weight>];
    [font-style: <style>];
}
```
各个参数对应解释如下：
|  参数   | 表头  |
|  :----  | :----:  |
| font-family: <YourWebFontName>  | 自定义字库名称（一般设置为所引入的字库名），后续样式规则中则通过该名称来引用该字库。 |
| src  | 设置字体的加载路径和格式，通过逗号分隔多个加载路径和格式 |
| srouce  | 字体的加载路径，可以是绝对或相对URL |
| format  | 字体的格式，主要用于浏览器识别，一般有以下几种——truetype,opentype,truetype-aat,embedded-opentype,avg等 |

> font-weight 和 font-style 和之前使用的是一致的。  
> src属性后还有一个 local(font name) 字段，表示从用户系统中加载字体，失败后才加载webfont。
```
src: local(font name), url("font_name.ttf")
```

### 字体格式
由于各浏览器所能识别的字体格式不尽相同，有如下字体格式：
#### TrueType格式(.ttf)
Windows和Mac上常见的字体格式，是一种原始格式，因此它并没有为网页进行优化处理。
#### OpenType格式(.otf)
以TrueType为基础，也是一种原始格式，但提供更多的功能。
#### Web Open Font格式(.woff)
针对网页进行特殊优化，因此是Web字体中最佳格式，它是一个开放的TrueType/OpenType的压缩版，同时支持元数据包的分离
#### Embedded Open Type格式(.eot)
IE专用字体格式，可以从TrueType格式创建此格式字体。
#### SVG格式(.svg)
基于SVG字体渲染的格式。  
> 这就意味着在@font-face中我们至少需要.woff,.eot两种格式字体，甚至还需要.svg等字体达到更多种浏览版本的支持。
为了使@font-face达到更多的浏览器支持，我们通常这样写：
```
 @font-face {
    font-family: 'YourWebFontName';
    src: url('YourWebFontName.eot'); /* IE9 Compat Modes */
    src: url('YourWebFontName.eot?#iefix') format('embedded-opentype'), /* IE6-IE8 */
             url('YourWebFontName.woff') format('woff'), /* Modern Browsers */
             url('YourWebFontName.ttf')  format('truetype'), /* Safari, Android, iOS */
             url('YourWebFontName.svg#YourWebFontName') format('svg'); /* Legacy iOS */
}
```

### Web字体获取与转化
到[Google Web Fonts](http://www.google.com/webfonts)和[Dafont.com](http://www.dafont.com/theme.php?cat=605)下载.ttf格式字体，然后通过[Font Squirrel](https://www.fontsquirrel.com/tools/webfont-generator)来生成.woff等格式的字体:
![](https://liangdo-top.oss-cn-shenzhen.aliyuncs.com/blog/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202021-08-08%20%E4%B8%8A%E5%8D%8812.08.23.png)
点击下载即可得到四种不同格式的字体文件了，然后在css中使用即可：
```
h2 {
  font-family: 'YourWebFontName'   
}
```
> 另外，如今使用iconfont之类的平台已经可以很方便获取到我们想要的字体文件了，但了解基础的css属性还是对我们写代码大有裨益的。