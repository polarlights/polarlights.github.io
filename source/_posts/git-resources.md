---
title: Git常用资源
date: 2015/01/30 20:00:00
category:
  - Tech
  - Git
tags:
  - Git
---

### 在工作目录中创建新仓库
进入新建的目录GitTest

```bash
$ git init
```

### 检查状态

```bash
$ git status
```

### 添加与提交
在当前工作目录中创建文件，这里我新建了`ReadMe.md`

### 添加变化

```bash
$ git add ReadMe.md
```

### 检查变化

```bash
$ git status
```
<!-- more -->

### 提交

```bash
$ git commit -m "Add ReadMe"
```

### 添加所有变化

比如添加所有的`.md`文件

```bash
$ git add '*.md'
```
### 提交所有变化

```bash
$ git commit -m 'Add all md files'
```

### 历史

```bash
$ git log
```

### 远程仓库

```bash
$ git remote add TestGit https://github.com/lay1010/TestGit.git
```
注意：按照官方教程，命令是`add origin`，而我之前已经误用过`origin`这个名字了，如果仍然git remote add origin，会出现fatal：remote origin already exists.

解决办法：将origin更换为其他名字，比如我现在更换为TestGit，后面的命令也相应地把origin替换为TestGit。感谢一楼评论`小明 曹`童鞋。

### 远程推送

```bash
$ git push -u TestGit master
```

### 远程拉入

```bash
$ git pull TestGit master
```

### 区别

```bash
$ git diff HEAD
```

### 阶段区别

```bash
$ git add testfamily/3.md
$ git diff --staged
```
`3.md`是刚刚添加的文件。


### 重置阶段

```bash
$ git reset testfamily/3.md
```
移除3.md，注意这里只是将3.md从staged状态移除。

### 撤销

```bash
$ git checkout -- 2.md
```
注意：checkout的用法不是很懂，这儿得问问。

### 分支出去

```bash
$ git branch clean_up
```
创建一个名叫`clean_up`的分支。

### 切换分支

```bash
$ git checkout clean_up
```
从master切换到了clean_up

### 移除所有东西

```bash
$ git rm '*.md'
```

### 提交分支变化

```bash
$ git commit -m "remove all things"
```

### 切换回master分支

```bash
$ git checkout master
```
### 准备合并分支

```bash
$ git merge clean_up
```

### 保持简洁（删除分支）

```bash
$ git branch -d clean_up
```
### 最终推送

```bash
$ git push
```
推送所有东西。

注意：完成所有步骤后发现blog里面所有的文件都消失了。原因我也不知道，看来看是得系统看书去。

### 退回到之前1天的版本

```bash
$ git log --before="1 days"
```


###资料链接
1. [Try Git](https://try.github.io/levels/1/challenges/1)
