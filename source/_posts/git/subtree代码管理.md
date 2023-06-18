---
title: 基于git subtree实现的代码管理
tags: ["git", "前端工程化"]
categories: git
date: 2020-12-23 14:42:09
---
### 基于git subtree实现的代码管理
随着公司业务的增长，各个项目多了起来，这时候需要对一些组件函数进行封装，以便在各个项目中使用，避免重复造轮子，相信这是大家常常会遇到的场景  
#### 应用场景
那么痛点来了，如果这个公共组件或方法也是一套正在开发的项目怎么办？如果是一个成熟的项目，那直接放npm私库中，需要的时候安装就好了。如果这套组件又在开发中，又不能和现有的具体项目混合在一起维护，有没有一个两全的方案呢？  
答案是有，git官方推荐的工具是git subtree，可以实现一个仓库作为其他仓库的子仓库  

#### 基本命令
git subtree涉及到的命令如下：
```
git subtree add   --prefix=<prefix> <commit>
git subtree add   --prefix=<prefix> <repository> <ref>
git subtree pull  --prefix=<prefix> <repository> <ref>
git subtree push  --prefix=<prefix> <repository> <ref>
git subtree merge --prefix=<prefix> <commit>
git subtree split --prefix=<prefix> [OPTIONS] [<commit>]
```

#### 例子
[这里](https://github.com/o4liangdu/subtree.git)是一个利用到git subtree管理项目的demo,主仓库是一个vue项目，引用了我的[一个时间处理库](https://github.com/o4liangdu/time-magic.git)的子项目  
关于指令和平时的git命令差不多，大家可以在工作中多使用两次就能熟练掌握了。  

这样，我们就能在多个项目中引用公共模块了，极大地提升了开发的灵活性。
