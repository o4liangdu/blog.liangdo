---
title: 以小见大，谈jquery(MVC)与vue(MVVM)的差别
date: 2021/06/08 12:54:22
tags: [jquery, vue, mvc, mvvm]
categories: vue
top: 3
toc: true
---
### 以小见大，谈jquery(MVC)与vue(MVVM)的差别
以jquery为代表的MVC框架与vue为代表的MVVM框架在前端领域代表了一个时代，反映了前端架构设计的思想转变，是值得我们去细品的

#### jquery为代表的MVC架构简介
MVC是一个常见的软件架构，是为了模块解耦诞生的，分为三个部分：  
+ 视图（View）：用户界面。
+ 控制器（Controller）：业务逻辑
+ 模型（Model）：数据保存  

三者的通信方式为单向的闭环，具体如下：
+ View 传送指令到 Controller
+ Controller 完成业务逻辑后，要求 Model 改变状态
+ Model 将新的数据发送到 View，用户得到反馈  

早期的backbone将Model和View的概念突出化，将Controller糅合进两者之间，体现了MVC的解耦思想  
jquery是使用选择器"\$"选取DOM对象，对其进行赋值、取值、事件绑定等操作，其实和原生的HTML的区别只在于可以更方便的选取和操作DOM对象，而数据和界面是在一起的。比如需要获取label标签的内容：$("lable").val();它还是依赖DOM元素的值。这里jquery充当了Controller的角色，被糅合进了view和model中

#### vue为代表的MVVM架构简介
MVVM改变了通信方向，ViewModel作为中间桥梁，和view、model之间的通信都是双向的。view 与 model不发生联系，都通过ViewModel传递   
view不部署任何业务逻辑，即没有任何主动性，而 所有逻辑都部署在ViewModel上  
vue专注于MVVM模型的ViewModel层,通过对数据的操作就可以完成对页面视图的渲染,视图的变化（用户操作）也能反馈到数据上  

#### 两者对比和各自的应用领域
上面的介绍从两个典型框架入手，说明了两种架构的不同点，网上有许多文章认为jquery基于view(DOM操作)不行了，要被淘汰了，我却不这么认为（比如新兴的Svelte也是基于DOM），虽然vue为代表的mvvm框架如今在前端领域大放异彩，但具体的应用害得根据实际应用来，脱离应用谈框架就是耍流氓。  
比如对于复杂数据操作的后台页面、表单填写页面、组件复用多的页面用vue是很合适的  
与之相对的，比如说一些html5的动画页面，一些需要js来操作页面样式的页面（直接操作DOM的场景），jquery的应用还是很广泛，也是很合适的。
