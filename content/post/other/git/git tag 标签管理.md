---
title: "Git Tag 标签管理"
date: 2019-03-06T19:48:36+08:00
categories:
- git
tags:
- git
keywords:
- git
- tag
---

跟大多数版本控制工具一样，git也有“标签”功能，一般用这个功能来标记发布节点,例如`v0.0.1`。

<!--more-->

> 示例命令以`v0.0.1`做为标签名

## 列出标签

```text
git tag 
```

命令会按照字母顺序列出当前仓库的所有标签。可以使用`-l`结合表达式搜索特定的标签。

```text
git tab -l v0.0*
```

## 创建本地标签

git提供两种主要的标签

### 轻量级标签

轻量级标签跟分枝一样，不会改变。它就是针对某个特定提交的指针。

创建轻量级标签

```text
git tag v0.0.1
```

### 带注释的标签

带注释的标签是git仓库中的特殊对象。它是一组校验和，包含标签名、创建者信息、日期，标签注释，GPG签名和验证信息。

创建带注释的标签

```text
git tag -a v0.0.1 -m "Release v0.0.1"
```

如果想要添加多行注释，可以去掉`-m`选项，git会自动打开文本编辑器，在文本编辑器中输入多行文本注释信息保存即可。

## 删除本地标签

删除标签可以使用`-d`选项

```text
git tag -d v0.0.1
```

## 提交标签到远程仓库

将本地**所有**标签同步到远程仓库

```text
git push origin --tags
```

提交指定标签

```text
git push origin v0.0.1
```

## 删除远程仓库标签

git版本大于`v1.7.0`

```text
git push origin --delete v0.0.1
```

git版本小于`v1.7.0`

```text
git push origin :refs/tags/v0.0.1
```
