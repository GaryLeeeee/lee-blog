---
title: 什么是JWT？
date: 2023-12-04 16:34:45
tags: [JWT,Token]
categories: [安全]
---

## 一、背景
在介绍JWT前，我们先了解一下Cookie、Session和Token，这几个概念我们会比较熟悉，它们是用于Web应用程序中管理用户状态和身份状态的技术。因为在Web应用程序中，Http的通信是无状态的，每个请求都是独立的，所以服务端无法确认当前访问者的身份信息（即每次请求是不是都是同一个人）。

Cookie、Session、Token有什么区别呢？：
* **Cookie**：保存在客户端的数据，会在下次请求时携带到Request中进行请求发送到服务端，服务端可以拿Cookie来做身份验证和做定制化处理（每个Cookie会有对应的单一域名，所以会出现跨域问题）
* **Session**：保存在服务端的数据，会生成一个sessionId给到客户端存储在Cookie中（也就是说Session要依赖Cookie），客户端下次请求就可以带上sessionId让服务端判断当前是哪个用户
* **Token**：不保存在哪一端的数据（除非要做退登删除等操作），由一个加密算法组成，客户端请求时只需要带上请求内容+签名给到服务端即可解析出请求参数+当前用户（签名可以防止篡改请求内容，对应生成的秘钥存在服务端，不可泄露）

从上面可以看到，用**Token**的方式是更优的，因为既能减少数据存储，也能避免数据被篡改。

## 二、什么是JWT？
JWT（JSON Web Token）是一种基于Token的认证授权机制，是目前业内流行的跨域认证解决方案。与Session不同的是，我们不需要存储Session信息，只需要通过JWT的加密算法进行生成和校验即可，减少了服务端的压力。

### 1、JWT组成
![](/images/security/JWT1.png)
JWT其实就是一串字符串，里面包含通过`.`拆分的三个Base64编码的内容：
* **Header**：定义该认证方式为JWT，包括加密算法以及Token类型
* **Payload**：需要传递的数据
* **Signature**：服务端通过Payload、Header和密钥生成的一个签名（算法用的是Header中的SHA256）

所以JWT一般长这样：`${Header}.${Payload}.${Signature}`，示例如下
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```
我们可以在[JSON Web Tokens](https://jwt.io/)进行解码得到Header、Payload、Signature，如下
![](/images/security/JWT2.png)

### 2、如何使用JWT？
![](/images/security/JWT3.png)
步骤如下：
1. 用户通过账号+密码登录成功
2. 服务端返回JWT给客户端
3. 客户端后续请求都带上JWT给服务端
4. 服务端校验JWT合法性和时效性

## 三、使用JWT要注意什么问题？
使用JWT虽然可以解决跨域认证问题，但是要注意以下问题：
* 建议客户端将JWT存在`localStorage（本地缓存）`而不是`Cookie`中，防止出现**CSRF攻击**
* 密钥要保存好，防止泄露出去
* Payload是明文的，需要避免存放隐私信息
* JWT尽量加上过期时间防止永久有效（可以在Payload的`exp`参数设置），当然也不宜过长

## 四、JWT的优缺点
### 1、JWT的优点
JWT的优点如下：
* **无状态**：JWT本身就包含了身份验证所需要的所有信息，所以服务端无需存储Session信息
* **避免CSRF攻击**：CSRF（Cross Site Request Forgery）叫做跨站请求伪造，也就是拿我们的身份信息去干坏事（如拿Cookie、Session去把你银行卡的钱转账出去），而JWT可以不依赖Cookie从而避免CSRF攻击
* **适合移动端应用**：传统的Cookie只适合用在Web端，但是JWT也可以用在移动端应用（只需要有地方可以存储JWT即可）
* **单点登录友好**：避免了Cookie可能出现的跨域问题、Session可能出现的多机器不同步Session信息问题，直接将JWT放在客户端就不会有这些问题了

### 2、JWT的缺点
因为JWT是无状态的，所以如果出现以下行为，单纯靠JWT我们是无法解决这些问题的：
* 用户被封禁
* 退出登录
* 修改密码
* 用户权限变更
* 用户被踢下线
* ...

在Session中，如果出现这些行为，我们只需要删除服务端对应的Session信息即可，但是如果用JWT的话，在失效之前它都是有效的。

所以我们一般有以下几种方案来解决：
* **存储JWT**：也就是把无状态改为有状态，在Redis/MySQL中存储一份JWT，但是缺点是增加了服务端的压力
* **黑名单机制**：相比**存储JWT**全量存储，该方案只需要把需要过期的JWT维护，大大减少了数据量
* **修改密钥**：对于不同用户使用不同的密钥，当出现上面行为时，把该用户密钥进行修改即可（缺点是需要额外存储不同用户对应的不同密钥，而且不适用于多端登录退登的问题，会导致一端退登另一端也需要重新登录）
* **过期时间缩短**：跟Redis一样，过期时间缩短了，重建缓存的次数就变多了，就可以让我们更快发现问题。但是会导致JWT无法长时间存储，需要用户经常登录

---

另外我们如何处理JWT的**续签**问题呢？主要有以下几种方案：
* **快过期时更新**：类似Session做法，服务端在客户端每次请求时会校验JWT有效期，如果快过期了就会更新新的JWT给客户端更新（缺点是快过期了才会更新）
* **每次都更新**：这种明显服务端压力会比较大，不推荐
* **双JWT机制**：从单JWT改为双JWT（即accessJWT+refreshJWT），当请求时会先携带有效期较短的accessJWT给服务端，如果accessJWT过期了则携带有效期较长的refreshJWT给服务端，然后服务端会返回新的accessJWT给客户端（缺点是实现较复杂，如要保证双JWT同时失效、会出现accessJWT短暂不可用等问题）

## 五、总结
JWT的优势在于**无状态**，但是如果我们要实现复杂场景（如上面提到的退登登问题），还是要存储JWT信息的。

当然我们也不是非得用JWT，用UUID、雪花算法等唯一性ID来做也是可以的，但是需要将该唯一标识存储起来（如存**Redis**中）。