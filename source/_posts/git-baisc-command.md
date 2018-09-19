---
title: Git基本操作回顾
date: 2018-09-19 17:14:55
tags: git
---

作为git最常用的几个命令`git status`、`git add`、`git commit`，我们每天可能都会写个数十遍。但是越是这种我们熟悉的操作，越容易存在一些我们忽略的细节。这篇文章就是用来记录下这些细节，记录我们常用命令中不常用的操作。

在git中编辑过某些文件之后，由于自上次提交后你对它们做了修改，git 将它们标记为已修改文件。 我们逐步将这些修改过的文件放入暂存区，然后提交所有暂存了的修改，如此反复。所以使用 git 时文件的生命周期如下：
![lifecycle.png](https://upload-images.jianshu.io/upload_images/1059465-6aee09f81493701f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

回顾完就进入正题
## git status
git status会有以下几种状态
```
$ Changes to be committed:
$ (use "git reset HEAD <file>..." to unstage)
```
表示已经在暂存区，等待添加到工作区。使用`git reset`命令可以将暂存区的内容移除。
```
$ Changes not staged for commit: 
$ (use "git add <file>..." to update what will be committed)
$ (use "git checkout -- <file>..." to discard changes in working directory)
```
有修改, 但是没有被添加到暂存区。使用`git add`命令可以将文件添加到暂存区，使用`git checkout`命令可以撤销文件修改。
```
$ Untracked files:
$ (use "git add <file>..." to include in what will be committed)
```
含有未跟踪文件, 即未纳入版本管理的文件。使用`git add`可以将文件放入暂存区。

---
## git add
添加文件到暂存区
```
git add file
```
添加多个文件到暂存区，空格隔开
```
git add file1 file2
```
使用通配符批量添加documentation目录下的所有txt后缀文件
```
git add documentation/*.txt
```
添加当前目录下的所有git-开头的shell文件
```
git add git-*.sh
```
将修改和以删除的文件添加到暂存区，不包括未被跟踪文件。
```
git add -u file
```
### git add .和git add -A(即git add --all)区别
一.版本导致的差别：
1.x版本：
（1）.git add -A可以提交未跟踪、修改和删除文件。
（2）.git add .可以提交未跟踪和修改文件，但是不处理删除文件。
2.x版本：
两者功能在提交类型方面是相同的。

二.所在目录不同导致的差异：
（1）.git add -A无论在哪个目录执行都会提交相应文件。
（2）.git add .只能够提交当前目录或者它后代目录下相应文件。

---

## git commit
当我们执行`git add`命令将文件放到暂存区之后，还需要提交这些暂存到工作区（仓库区），从暂存区->工作区，的工作就是`git commmit`来做的。
```shell
# 提交暂存区到仓库区，message为提交信息
git commit -m [message]
# 提交可以指定文件
git commit [file1] [file2] ... -m [message]
```
常用的commit扩展命令
```shell
# 提交时显示所有diff信息
git commit -v
# 使用一次新的commit，替代上一次提交，如果代码没有任何新变化，则用来改写上一次commit的提交信息
git commit --amend
# 重做上一次commit，并包括指定文件的新变化
git commit --amend [file1] [file2]
```
以上三条如果不带-m [message]将会在vim的编辑器中添加提交信息。

如果你感觉没有`git add`,`git commit`有点麻烦，想直接将修改到工作区，可以用另外一个命令。
```shell
# 会将上次commit之后的变化，直接添加到工作区
git commit -a -m [message]
```
---
## git rm
```
rm file
```
**删除位置**：相当于手动右击点删除，只删除了工作区的文件。
**git status**：`Changes not staged for commit:`
**恢复**：直接用git checkout -- file就可以。
```
git rm file
```
它等价于`rm file + git add file`
**删除位置**：相当于不仅删除了文件，而且还添加到了暂存区。
**git status**：`Changes to be committed:`。
**恢复**：先git reset，去掉暂存区修改，然后再git checkout -- file，恢复文件。
```
git rm --cached file
```
**删除位置**：从暂存区移除，不删除文件。
**git status**：`Changes to be committed:`，`Untracked files:`
**恢复**：git add file
