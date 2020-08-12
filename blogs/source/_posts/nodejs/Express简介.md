---
title: Express简介
tags:
categories:
- nodejs
---

### Express 是一个自身功能极简，完全由路由和中间件构成的一个web开发框架
中间件: 就是匹配路由之前和匹配路由之后做的一系列操作.
中间件是一个函数，可以访问请求对象，响应对象，和web应用中处理请求响应循环流程中的中间件，一般被命名为next的变量


中间件功能包括:
 * 执行任何代码
 * 修改请求和响应对象
 * 终结请求，响应循环
 * 调用堆栈中的下一个中间件
 
如果get，post回调函数中，没有next参数，那么就匹配第一个路由，就不会往下匹配. 如果往下匹配必须写next.

Express 应用可使用如下几种中间件:
 * 1. 应用级中间件
 * 2. 内置中间件
 * 3. 路由级中间件
 * 4. 错误处理中间件
 * 5. 第三方中间件

### 例子:
```
var express=require('express'); // 引入
var app=new express();			// 实例化
/* 1. 应用级中间件app.use(), 匹配所有路由,就是在匹配其它路由之前都需要执行的操作，多用在权限判断, 没登陆之前是否可登陆 或 是否有权限访问某些网页
* 下面中间件: 表示匹配任何路由, 如再匹配其它路由之前打印下时间
*/
app.use(function(req, res, next){
	console.log(new Date());
	// next();		// 缺少next, 匹配到此路由并打印出时间后浏览器头部观察到转圈圈不再往下匹配.
	if(req.session != Null){
		next();
	}else
	{
		res.redir("/login");
	}
})

/* 2. 内置中间件，托管静态页面, 匹配路由之前, 看public下面有没有匹配的文件, 如果有则返回，没有就继续向下匹配*/
// 项目部分目录结构: project_name/public/css/style.css
// 第一种访问方式: http://127.0.0.1:3000/css/style.css
app.use(express.static('public'));
// 第二种访问方式: http://127.0.0.1:3000/static/css/style.css
app.use("/static", express.static('public'));

/* 3. 路由中间件app.get(): 匹配某个路由 */
app.get("/", function(req, res, next){
	console.log("您好express");
	next();						// 缺少next(), 浏览器会一直转圈圈卡着等待服务器继续返回.
});
app.get("/", function(req, res){
	res.send("您好express");	// 匹配到此路由后, res.send()后不需要再next, 浏览器会结束响应.
});

/* 1. 应用级中间件app.use()*/
app.use("/news", function(req, res, next){
	console.log("应用级中间件匹配news");
	next();
})
app.get("/news", function(req, res){
	res.send("news");
});

/* 4. 错误处理中间件, 如果上面路由都没有匹配到，404*/
app.use(function(req, res){
	res.status(404).send("这是404，路由没有匹配到");
})

/* 5. 第三方中间件就是官方或者其他人好的中间件，访问https://npmjs.com搜索下载 */ 
/* 
* 不用中间件获取post提交数据: req.on(); 用中间件body-parser获取post提交的数据.
* 一: 安装: npm install body-parser --save
* 二: 引用: var bodyParser = require('body-parser');
* 三：设置中间件
*	// parse application/x-www-form-urlencoded
*	app.use(bodyParser.urlencoded({extended: false}))
*	//parse application/json
*	app.use(bodyParser.json())
* 四: req.body 获取post提交的数据
*/

// 二：引入body-parser中间件
var bodyParser = require('body-parser');
// parse application/x-www-form-urlencoded
// 三：配置body-parser中间件
app.use(bodyParser.urlencoded({extended: false}))
//parse application/json
app.use(bodyParser.json())

// ejs, 1.安装(package.json)，2.如下配置 ejs引擎
// 默认ejs加载的是views下面的视图, 如login.ejs
app.set('view engine', 'ejs');
app.get('/login', function(req, res){
	res.render('login');
})
/*	login.ejs文件路径: project_name/views/login.ejs, 内容含有如下表单
* <form action="doLogin" method="post">
*	用户名: <input type="text" name="username"/><br/>
*	密  码: <input type="password" name="password"><br/>
*	<input type="submit" value="登陆"/>
* </form>
*/
// 四：使用req.body 获取post提交的数据 
app.post("/doLogin", function(req, res){
	console.log(req.body);
})
```