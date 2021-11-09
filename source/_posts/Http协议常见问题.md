---
title: Http协议常见问题
date: 2021-11-07 23:30:48
tags: Http
categories: Http
---

### 1.参数值传yyyy-MM-dd报错(400)
**解决方案**：改成yyyy/MM/dd
![Http接口传时间格式错误g](/images/Http接口传时间格式错误.png)
![Http接口传时间格式正确](/images/Http接口传时间格式正确.png)


### 2.参数值直接传json报错(400)
**例子**：{"id":3,"name":"gary"}
**原因**：Http Get和Post请求不能传包含`{`、`}`等这类特殊字符
**解决方案**：需要对特殊字符进行转义，如`"`转成`%22`,`{`转成`%7b`等
**在线转义**：http://www.jsons.cn/urlencode/
![Http接口参数转义](/images/Http接口参数转义.png)