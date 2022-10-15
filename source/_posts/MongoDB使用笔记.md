---
title: MongoDB使用笔记
date: 2022-02-21 21:37:51
tags: [MongoDB]
categories: [MongoDB]
---
## 一、安装
https://www.runoob.com/mongodb/mongodb-window-install.html

## 二、可视化工具
下载链接：https://robomongo.org/download
使用指引：https://blog.csdn.net/wangmx1993328/article/details/82628805

## 三、常见指令(随用随更新)
### 1.查询文档
如查询collection为user的uid为123456的文档，则
完整：```db.getCollection("user").find({"user":123456})```
简化：```db.user.find({"user":123456})```

## 四、常见问题
### 1.linux如何连接mongo？
* a.下载mongo指令
```wget https://fastdl.mongodb.org/linux/mongodb-shell-linux-x86_64-ubuntu1604-4.0.28.tgz```
* b.解压
```tar zxvf mongodb-shell-linux-x86_64-ubuntu1604-4.0.28.tgz```
* c.跳转到指令目录
```cd mongodb-linux-x86_64-ubuntu1604-4.0.28/bin```
* d.连接
```mongo --host ${host} --port ${port} -u ${username} -p ${password}" ```

### 2.linux如何导出mongo数据？
* a.下载mongoexport指令
```wget https://fastdl.mongodb.org/tools/db/mongodb-database-tools-rhel70-x86_64-100.3.1.tgz```
* b.解压
```tar zxvf mongodb-database-tools-rhel70-x86_64-100.3.1.tgz```
* c.跳转到指令目录
```cd mongodb-database-tools-rhel70-x86_64-100.3.1/bin```
* d.导出数据
```mongoexport -d ${database} -c ${collection} -q ${queryJson} --out ${exportFilePath}```
  * **导出类型**：`--type`默认是`json`，对应的`--out`需要是`.json`后缀，`--type`也可以是`csv`，对应的`--out`需要是`.csv`后缀.csv类型必须指定导出字段`--fields`
  * **查询语句**：必须是json语句，且外层用单引号，如`-q '{"uid":123456}''`