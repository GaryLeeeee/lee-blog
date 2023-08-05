---
title: RabbitMQ安装笔记
date: 2021-11-05 21:07:13
tags: [RabbitMQ,工具安装]
categories: [RabbitMQ]
---

# windows10安装
## 下载并安装erlang
* 原因：RabbitMQ服务端代码是使用并发式语言Erlang编写的，安装Rabbit MQ的前提是安装Erlang。
* 下载链接：https://www.erlang.org/downloads

![erlang下载](/images/erlang下载.png)

* 安装
* 配置环境变量
    * 第一步：此电脑->鼠标邮件"属性"->高级系统设置->环境变量->系统变量"新建"
        * 变量名：ERLANG_HOME
        * 变量值：erlang安装目录(我的是`G:\software\erlang`)
![erlang环境变量](/images/erlang环境变量.png)
    * 第二步：系统变量双击"path"->新建->添加`%ERLANG_HOME%\bin`
![erlang环境变量2](/images/erlang环境变量2.png)
![erlang环境变量3](/images/erlang环境变量3.png)
   
* win+R->输入cmd->输入erl，显示版本号就说明安装成功了
![erlang查看版本](/images/erlang查看版本.png)

## 下载并安装RabbitMQ
* 下载地址：http://www.rabbitmq.com/download.html
![RabbitMQ下载](/images/RabbitMQ下载.png)
  
* win+R->输入cmd->cd到RabbitMQ的sbin目录->输入`rabbitmq-plugins enable rabbitmq_management`
![RabbitMQ安装](/images/RabbitMQ安装.png)  
* 打开sbin目录->双击`rabbitmq-server.bat`
![RabbitMQ启动](/images/RabbitMQ启动.png)
* 过一会出现这个页面，说明启动成功了，这时候可访问`http://localhost:15672`
![RabbitMQ启动成功](/images/RabbitMQ启动成功.png)
* 这是RabbitMQ的后台，username和password默认是`guest`
![RabbitMQ后台](/images/RabbitMQ后台.png)
* end