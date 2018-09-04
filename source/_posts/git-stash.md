---
title: 使用git stash储存和恢复进度
date: 2018-09-04 14:53:34
tags: git
---

当我们正在当前项目处理一些事情时，有一个需求插进来，使得我们要在别的分支做一些工作。切换分支之前当前任务是需要保存的，但我们并没有完成一个完整的任务，直接`commit`显得不合适，这时就可以使用`git stash`命令。stash是储藏的意思，该命令的作用也可以理解为先将当前的修改储藏起来，等我们在其他分支做完必要工作之后可以再回到储藏时的状态。

`git stash`大致可以分为储存和恢复这两步。

## 储存
储藏当前进度有两条命令：
```
git stash
```
保存当前工作进度，会把暂存区和工作区的改动都保存起来，再次运行`git status`会发现当前工作区是干净的。
```
git stash save "commit message"
```
是`git stash`的完整描述，可以为本次保存添加说明。

## 恢复
```zsh
git stash list
```
查看当前保存进度，进度保存可以有多个。
```zsh
git stash apply
```
恢复最近保存的进度，不会删除stash内容
```zsh
git stash apply stash@{0}
```
如果有多个stash，恢复某一个，按时间倒叙排列
```zsh
git stash pop
```
会恢复最新保存的工作进度，并将恢复的工作进度从存储的工作进度列表中清除。
```zsh
git stash drop [stash_id]
```
删除某一个存储的进度
```zsh
git stash clear #删除所有储存进度
```
删除所有存储进度

