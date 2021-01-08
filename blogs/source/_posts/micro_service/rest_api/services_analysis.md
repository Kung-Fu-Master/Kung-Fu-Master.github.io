---
title: 1.services analysis
tags: kubernetes
categories:
- microService
- REST-API
---
![](architecture.png)
# API Gateway

# Zookeeper 服务的注册和发现

# 服务API
 * REST
 * Thrift
 * Dubbo - 基于kv的存储来进行服务的发布和订阅

## REST API
![](REST_FUL_API.png)  
 * REST，即Representational State Transfer的缩写,中文是"表现层状态转化"。
通俗来讲就是：资源在网络中以某种表现形式进行状态转移。(再通俗来说，就是通过HTTP请求服务器上的某资源，使该资源copy了一份到服务请求方那去了(get动作)。
Representational：某种表现形式，比如用JSON，XML，JPEG等；HTTP请求的头信息中用Accept和Content-Type字段指定，这两个字段才是对"表现形式"的描述。
State Transfer：状态变化。通过HTTP动词（GET,POST,DELETE,DETC）实现。
互联网通信协议HTTP协议，是一个无状态协议。**这意味着，所有的状态都保存在服务器端。
**因此，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生"状态转化"（State Transfer）。
HTTP协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE。
它们分别对应四种基本操作：GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT用来更新资源，DELETE用来删除资源。

 * REST是由谁提出来的:
Roy Thomes Fielding在他2000年的博士论文中提出REST架构模式，他是HTTP协议(v1.0和v1.1)的主要设计者、Apache服务器作者之一、Apache基金会第一任主席。

 * 什么是REST ful API
基于REST构建的API就是Restful风格。

 * 为什么产生了这种架构模式
传统的那种JSP前后端耦合在一起的网页模式我们称之为“上古时期”网页，这种模式弊端很多。
近年来，随着移动技术的发展，各种移动端设备层出不穷，RESTful可以通过一套统一的接口为 Web，iOS和Android提供服务。
另外对于广大平台来说，比如新浪微博开放平台，微信公共平台等，它们不需要有显式的前端，只需要一套提供服务的接口，于是RESTful更是它们最好的选择。

 * 如何设计规范的REST ful API接口
  - [RESETful API 设计规范](https://godruoyi.com/posts/the-resetful-api-design-specification)
  -  [RESTful API 最佳实践](http://www.ruanyifeng.com/blog/2018/10/restful-api-best-practices.html)

## 单点登陆系统 慕课网有免费课程
访问单点登陆系统，校验拿到的token或者称为tickets(票据)是不是正确的, 然后使用这个东西去换取用户的具体信息, 再存储到当前的服务里面



