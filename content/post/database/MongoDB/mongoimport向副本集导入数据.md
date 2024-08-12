---
title: "mongoimport向副本集导入数据"
date: 2024-08-12T09:20:51+08:00
lastmod: 2024-08-12T09:20:51+08:00

categories:
  - MongoDB
tags:
  - mongoimport
keywords: 
  - mongoimport

# 开启数学公式渲染，可选值： mathjax, katex
#math: mathjax

# 开启各种图渲染，如流程图、时序图、类图等
#mermaid: true
---

mongoimport向副本集导入数据

<!--more-->

## 使用 --host 选项

```bash
# 格式
mongoimport --host=${RS-NAME}/${HOST-1}:${PORT-1},${HOST-2}:${PORT-2},${HOS3-1}:${POR3-1} --db=${DATABASE} --username=${USERNAME} --password=${PASSWORD} --authenticationDatabase=admin --file=/PATH/TO/FILE.json 
```

## 使用 --uri 选项

```bash
mongoimport --uri="mongodb://${USERNAME}:${PASSWORD} @${HOST-1}:${PORT-1},${HOST-2}:${PORT-2},${HOS3-1}:${POR3-1}/${DATABASE}?authSource=admin&replicaSet=${RS-NAME}" --file=/PATH/TO/FILE.json 
```