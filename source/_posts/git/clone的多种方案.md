---
title: clone的多种方案
tags: ["git", "前端工程化"]
categories: git
date: 2022-12-27 11:22:03
---
### clone的多种方案

当我们拉取项目时，会用到git clone的命令，如果我们默认使用git clone 项目地址 的方式去拉取远程仓库的话，我们会得到一个包含完整commit记录的项目。这样会有很明显的缺点：项目可能很大，很多commit纪录会导致拉取代码的过程异常漫长，且占有相当大的磁盘空间

因此，学会更多clone的技巧，是很有必要的

#### --depth 1
我们可以在克隆时指定--depth 1，--depth后面的阿拉伯数字代表克隆仓库的最新几个版本，为1代表只克隆远程仓库的最新的一个版本：
```
git clone --depth 1 https://github.com/dogescript/xxxxxxx.git
```
我们可以在.zshrc进行alias的设置：
```
alias 'gsc'='git clone --depth 1'
```
这样，我们就能通过gsc 项目地址 的方式克隆项目了


注意：这样只会把默认分支clone下来，其他远程分支并不在本地，所以这种情况下，需要用如下方法拉取其他分支：
```
git remote set-branches origin 'remote_branch_name'
git fetch --depth 1 origin remote_branch_name
git checkout remote_branch_name
```

#### degit

git clone 方法包括 --depth=1 都有个缺点，就是会带一个 git 环境，如果我们是开始一个新项目，需求是指向获取模板代而已，这时我们就可以使用 degit 工具来去除 git:
全局安装：
```
npm install -g degit
```
克隆repo并命名：
```
degit vitest-dev/vitest my_project_name
```
