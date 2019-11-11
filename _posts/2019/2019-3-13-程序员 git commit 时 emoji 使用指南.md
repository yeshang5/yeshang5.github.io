---
layout: post
title: '程序员git commit时emoji指南'
date: 2019-2-20
author: 白皓
cover: ''
tags: Git
---
  
> 执行 git commit 时使用 emoji 为本次提交打上一个 “标签”, 使得此次 commit 的主要工作得以凸现，也能够使得其在整个提交历史中易于区分与查找。

commit 格式
git commit 时，提交信息遵循以下格式：
```java
:emoji1: :emoji2: 主题

提交信息主体

Ref <###>
```

初次提交示例：
```java
git commit -m ":tada: Initialize Repo"
```


详情emoji代码查看此处(https://github.com/yeshang5/emoji-cheat-sheet)