---
title: "Git删除文件及历史的方法"
date: 2021-04-07T15:16:29+08:00
categories:
- git
tags:
- git
- 删除
keywords:
- git
- 删除
---

在日常开发过程中，如果不小心将一个敏感文件，或者一个不必要的文件提交到git仓库时，可能会引起不良的后果。一般情况下可以使用 `git rm` 删除这个文件，但是 git 历史中依然保存了这个文件的历史版本。如果要彻底删除它，可以使用 命令来彻底删除。

<!--more-->

## 删除缓存

```text
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch password.txt' --prune-empty --tag-name-filter cat -- --all
```

命令解释

* **filter-branch** : 让git重写每一个分支
* **--force** : 遇到冲突也强制执行
* **--index-filter** : 指定重写的时候执行什么命令，在这里是 `git rm --cached --ignore-unmatch password.txt` ，让git删除掉缓存的文件。
* **--prune-empty** : 如果重写导致commit变成了空（比如修改的文件全部被删除），那么删除掉这个commit。
* **--tag-name-filter** : 重写标签名称的过滤器，表示对每一个tag如何处理，当前的tag会标准输入的形式输入给命令，命令执行的标准输出值做为新的tag标记。这里的 `cat` 就表示保持tag名不变。
* **--** : 分割符
* **--all** : rev-list，表示针对所有的文件

## 提交

```text
git push -f --all
```