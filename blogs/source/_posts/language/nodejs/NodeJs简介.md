---
title: NodeJs简介
tags:
categories:
- language
- nodejs
---

### bilibili网址:[https://www.bilibili.com/video/BV1FJ411Q7fi](https://www.bilibili.com/video/BV1FJ411Q7fi)

## node.js 网站

> 1. [node.js官方网站](https://nodejs.org/)
> 2. [node.js中文网](http://nodejs.cn/)
> 3. [node.js 中文社区](https://cnodejs.org/)

## NodeJs简介
 * 开发工具: WebStorm, VScode.
 * Node.js是一个javascript运行环境，它让JavaScript可以开发后端程序.
 * Nodejs是基于V8 JS引擎，V8 JS引擎是Google发布的开源JavaScript引擎,本身就是用于Chrome浏览器的JS解析部分.
 * Ryan Dahl把V8 JS引擎搬到了服务器上，用来做服务器的软件.
 * 基于 node.js 可以开发控制台程序（命令行程序、CLI程序）、桌面应用程序（GUI）（借助 node-webkit、electron 等框架实现）、Web 应用程序（网站）

 npm 官网: [https://www.npmjs.com/](https://www.npmjs.com/)

 * NodeJs语法完全是JS语法，打破了过去JavaScript只能在浏览器中运行的局面，前后端编程环境统一.

 * node.js 全栈开发技术栈: MEAN - MongoDB Express Angular Node.js

### node.js 有哪些特点？

> 1. 事件驱动(当事件被触发时，执行传递过去的回调函数)
> 2. 非阻塞 I/O 模型（当执行I/O操作时，不会阻塞线程）磁盘I/O(文件I/O)，网络I/O(当向网络发送一些数据或接受一些数据也称为I/O)
> 3. 单线程(V8引擎只有堆deep和一个调用栈stack, 之所以单线程又是非阻塞, 原因是在NodeJs底层开辟一个异步操作来做如写文件等,JS代码依然是单线程, Nodejs底层帮我们把文件写完后，把回调函数放到队列里, NodeJs主线程执行完栈里面函数后，发现队列里有回调函数，就取出来到栈里再执行)
	在线动画演示：http://latentflip.com/loupe
> 4. 拥有世界最大的开源库生态系统 —— npm。

### NodeJs超强的高并发能力
Java、PHP、.net 等服务器端语言中，会为每个客户端创建一个新线程，每个线程需要耗费约2MB内存.  

理论上，一个8GB内存的服务器可以同时连接的最大用户数为4000个左右，如果要实现进一步的高并发就需要增加服务器的数量，硬件成本就会上升.

Node.js不为每个客户的连接创建一个新的线程，而仅仅使用一个线程，当用户连接就触发一个内部事件，通过非阻塞I/O、事件驱动机制

让Node.js程序宏观上也是并行的. 使用Node.js, 一个8GB内存的服务器，可以同时处理超过4万用户的连接.

总结: 同样的服务器配置，Node.js的并发量将近是传统的后端语言的10倍.

### 实现高性能服务器
V8 JavaScript并不局限于在浏览器中运行，Node.js将其用在了服务器中.

该引擎使用C++语言开发的一种高性能JavaScript引擎，引擎内部使用一种全新的编译技术, 解析并执行JavaScript脚本语言

意味着用JavaScript编写的语言可以有与低端c语言非常相近的执行效率,所以NodeJs是可以实现高性能服务器.


#### 非阻塞

``` js
var fs = require('fs');
console.log('1');
fs.readFile('my.json', function(err, data){
	console.log('2');
})
console.log('3');
输出：
1
3
2
```

#### 回调函数

``` js
var fs=require('fs');
function getmsg(callback){
	fs.readFile('my.json', function(err, data){
		callback(data);
	})
})

getmsg(function(data_result){
	console.log(data_result.toString());
})

```




