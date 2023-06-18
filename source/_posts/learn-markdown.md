---
title: 常用markdown语法
mathjax: true
categories: markdown
tags: ["markdown"]
top: 2
date: 2019-09-30 14:42:00
---

## 这是一个二级标题，以#数量来表示

### 引用
>
> 这是一段引用  
> 换行
>> 二级引用
>>> 三级引用

### 图片、链接

[1]:
    这是段注释
![alt img](../images/lufei.jpg)  
{% asset_img lufei.jpg This is an example image %}  
这个链接用 1 作为网址变量 [demo][1].
然后在文档的结尾为变量赋值（网址）  
<img src="../images/lufei.jpg" width="80px">
<img src="lufei.jpg" width="80px">

[这是md的链接](https://liangdo-top.oss-cn-shenzhen.aliyuncs.com/blog/learn-markdown.md)
<a href="https://liangdo-top.oss-cn-shenzhen.aliyuncs.com/blog/learn-markdown.md" download="markdown.md">此md的链接</a>

### 粗体与斜体

**这是粗体**  
*这是斜体*

### 分割线、删除线

***
~~删除的东西~~  
<u>带下划线文本</u>

### 脚注

效果：[^要注明的文本]
[^要注明的文本]:说明文本

### 列表

+ 第一项
+ 第二项
+ 第三项

1. 第一项
2. 第二项
3. 第三项

1. 第一项：
    + 第一项嵌套的第一个元素
    + 第一项嵌套的第二个元素
2. 第二项：
    + 第二项嵌套的第一个元素
    + 第二项嵌套的第二个元素

### 代码

    console.log("hello")
    print("halo")
    if() {
        do()
    }
`printf()`

```js
console.log("hello")
print("halo")
if() {
    do()
}
```

### 表格

Markdown 制作表格使用 | 来分隔不同的单元格，使用 - 来分隔表头和其他行。  
-: 设置内容和标题栏居右对齐。  
:- 设置内容和标题栏居左对齐。  
:-: 设置内容和标题栏居中对齐。  
|  表头  | 表头  |
|  ----: | :----:|
| 单元格  | 单元格 |
| 单元格  | 单元格 |

### 公式

$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix}
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$
