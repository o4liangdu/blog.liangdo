---
title: 如何写一个babel插件
date: 2021/04/21 14:18:31
tags: [webpack, nodejs, js]
categories: [webpack, 实战项目]
top: 3
toc: true
---
### 如何写一个babel插件
上篇文章{% post_link webpack/如何写一个loader %}中我们基于字符串完成了一个webpack loader，这篇文章将会基于AST实现一个babel插件  

涉及到的概念有[babel](https://zhuanlan.zhihu.com/p/43249121)、[抽象语法树AST](https://zhuanlan.zhihu.com/p/102385477)，这里就不一一详细介绍了，相关了解的链接已经附上，下面只会介绍基于babel编写插件用到的相关原理
> 可以通过[这个网站](https://astexplorer.net/)在线生成你的代码的ast  

#### 相关原理
首先说下各个模块的职责，babel做了一件什么事呢？简单来说，就是：
1. code(字符串形式代码)-> tokens(令牌流)-> AST（抽象语法树），这块是babel-core的核心API transform做的工作，将字符串转成了AST

2. 生成AST以后对其进行遍历，在此过程中对节点进行添加、更新及移除等操作,这是插件进行的主要工作

3. 最后就是深度优先遍历整个 AST，然后构建可以表示转换后代码的字符串，这个是@babel/generator已经为我们做好的工作

#### 实践
[这里](https://github.com/o4liangdu/trans-babel-plugin.git)是我基于babel实现的一个plugin的项目，主要功能如下：   
1. let,const声明 -> var声明
2. 箭头函数 -> 普通函数

当访问AST节点时，是通过访问者模式(vistor)去获取的，简单来说这是一个对象，看下面例子：
```js
const MyVisitor = {
    Identifier (path) {
        console.log("Visiting: " + path.node.name); 
    }
};
```
这样，在遍历的时候，每当在树中遇见一个 Identifier 的时候会调用 Identifier() 方法,path是表示两个节点之间连接的对象  
Babel的插件模块需要我们暴露一个function,function内返回visitor对象：
```js
module.exports = function({ types: t}) {
    return {
        // 访问者
        visitor: {
           
        }
    }
}
```
path.node.kind代表了声明的类型，将let和const换成var即可；  
由于箭头函数对应的AST节点是ArrowFunctionExpression，而普通函数对应的节点是FunctionExpression，同样地通过相应api替换：
```js
VariableDeclaration(path) {
    const node = path.node;
    // 判断节点kind属性是let或者const，转为var
    [ 'let' , 'const' ].includes(node.kind) && (node.kind = "var")
},
ArrowFunctionExpression(path) {
    // 该路径对应的节点信息
    let {id,params,body,generator,async} = path.node;
    if(!t.isBlockStatement(body)) {
        const node = t.returnStatement([node])
        body = t.blockStatement([node])
    }
    // 进行节点替换
    path.replaceWith(t.functionExpression(id,params,body,generator,async))
}
```
相比于之前通过字符串进行替换，这个方法更为强大，也不会像字符串那样容易出错，这里面api细节很多，大家可以多多尝试。
