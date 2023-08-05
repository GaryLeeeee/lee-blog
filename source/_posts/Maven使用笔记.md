---
title: Maven使用笔记
date: 2021-11-07 23:14:36
tags: [Maven]
categories: [Maven]
---


## 常见问题
### 长时间卡在"Resolving Maven dependencies"
解决方案：`Setting`->`Build,Execution,Deployment`->`Build Tools`->`Maven`->`Importing`->`VM options for importer`改为`-Xms1024m -Xmx2048m`
![maven长时间卡住resolving](/images/maven长时间卡住resolving.png)