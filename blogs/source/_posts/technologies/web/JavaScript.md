---
title: JavaScript
tags:
categories:
- technologies
- web前端
---

### ES6 新特性

//※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
### ES6 let 块作用域
``` javascript
if(true){
    let a = 123;  // let 是块作用域，只能再if作用域使用
    console.log(a);
}
``` 
//※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
### ES6 模板语法
``` javascript
var name = "张三";
var age = 20;
console.log(`${name}的年龄是${age}`);  // 不是引号''，而是键盘左上角的`
```
//※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
### ES6 属性的简写
``` javascript
var name = "zhangsan";
//1.
var app1 = {
    name:name
}
//2.
var app2 = {
    "name":name
}
//3. 简写
var app3 = {
    name
}
console.log(app1.name);
console.log(app2.name);
console.log(app3.name);
``` 
//※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
### ES6 buffer简写
``` javascript
var name = "李四";
var app4 = {
    name,
    run:function(){
        console.log(`${this.name}在跑步`);
    }
}
var app5 = {
    name,
    run(){
        console.log(`${this.name}在跑步`);
    }
}
app5.run()
```
//※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
### 箭头函数
``` javascript
setTimeout(function(){
    console.log("执行");
},1000);
setTimeout(()=>{
    console.log("执行");
}, 1000);

// 回调函数获取异步函数里数据
//1.
function getData1(){
    //ajax, 异步
    setTimeout(()=>{
        var name1 = "张三1";
    },1000);
    return name1;   // 执行会出错，找不到name1变量
}
//console.log(getData1());  // 出错这是获取不到name1的
//2.
function getData2(callback){
    setTimeout(()=>{
       var name2 = "张三2";
       callback(name2);
    });
}
getData2((aaa)=>{
    console.log(aaa);
});
//3. Promise来处理异步  resolve 成功的回调函数   reject失败的回调函数
var p1 = new Promise((resolve, reject)=>{
    setTimeout(()=>{
        var name3 = "张三3";
        resolve(name3);
    },1000);
});
p1.then((data)=>{
   console.log(data);
});
//4. 提取一下Promise里参数
function getData3(resolve, reject){
    setTimeout(()=>{
        var name4 = "张三4";
        resolve(name4);
    }, 1000);
}
var p2 = new Promise(getData3);
p2.then((data)=>{
    console.log(data);
})
```
//※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
### async声明异步方法, await等待异步方法执行完成
``` javascript
//1.
async function test1(){     // 函数定义为async，返回Promise { '您好！' }
    return "您好！";
}
// console.log(await test());   // 出错， await必须用在async函数方法中
//2.
async function test2(){
    return new Promise((resolve, reject)=>{
        setTimeout(()=>{
            var name5 = "张三5";
            resolve(name5)
        }, 1000);
    });
}
async function output(){
    var data = await test2();    // 如果不加await，返回是 Promise { '您好！' }
    console.log(data);          // 返回 您好！
}
output();

// 判断一个资源到底是目录还是文件，此方法里有异步函数，需要加上async标签，再用Promise来获取异步返回，最后再用await来调用等待此异步方法执行完
async function isDir(path){
    return new Promise((resolve, reject)=>{
        fs.stat(path, (err, states)=>{
            if(err){
                console.log(err);
                reject(err);
                return ;
            }
            if(states.isDirectory()){
                resolve(true);
            }else{
                resolve(false);
            }
        });
    });
}
// 获取所有资源
function getDir(){          // 此方法不需要加async, await只在它所在的函数上加上async就可以了，更外层的函数不需要加上asycn
    var path = "./testfs";
    var dirArr = [];
    fs.readdir(path, async (err, data)=>{   //回调函数需要加async，因为此函数里需要用await
        if(err){
            console.log(err);
            return ;
        }
        for(var i = 0; i < data.length; i++){
            if(await isDir(path + "/" +data[i])){
                dirArr.push(data[i]);
            }
        }
        console.log(dirArr);    // 输出 [ 'fs1', 'fs2' ]
    });
}
getDir();
//※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
```


### JSON.parse(text[, reviver])
	参数说明：
	text:
		必需， 一个有效的 JSON 字符串。
	reviver: 
		可选，一个转换结果的函数， 将为对象的每个成员调用此函数。
	返回值：
		返回给定 JSON 字符串转换后的对象。

``` html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>title)</title>
</head>
<body>
<p id="demo">测试JSP函数库JSON.parse()</p>
<script>
const json_str = '{"result":true, "count":42}';
const json = { "name":"MYZ" };

// 用来解析JSON字符串，构造由字符串描述的JavaScript值或对象
const obj = JSON.parse(json_str);
console.log(obj.count);		// expected output: 42
console.log(obj.result);	// expected output: true

console.log(obj);		// > Object { result: true, count: 42 }
console.log(json);		// > Object { age: 23 }
console.log(typeof(json_str));			// > "string"
console.log(typeof(json));				// > "object"
console.log(typeof(obj) === typeof(json));	// > true

for(x in json){
  console.log(x);		// > "name"
  console.log(json[x]);		// > "MYZ"
}

JSON.parse('{"result":true, "count":42}', function(k, v) {
  console.log( k );		// 输出当前三个属性，最后一个为 "", > "result", > "count", > ""
  return v;			// 返回修改的值
});

JSON.parse('{"1": 7, "2": 2, "3": {"4": 4, "5": {"6": 6}}}', function(k, v) {
  console.log( k );		// 输出当前属性，最后一个为 "" > "1", > "2", > "4", > "6", > "5", > "3", > ""
  return v;			// 返回修改的值
});

</script>
</body>
</html>
```

### JSON.stringify(value[, replacer [, space]])
	value:
		将要序列化成 一个 JSON 字符串的值。
	replacer 可选:
		如果该参数是一个函数，则在序列化过程中，被序列化的值的每个属性都会经过该函数的转换和处理；
		如果该参数是一个数组，则只有包含在这个数组中的属性名才会被序列化到最终的 JSON 字符串中；
		如果该参数为 null 或者未提供，则对象所有的属性都会被序列化；
		关于该参数更详细的解释和示例，请参考使用原生的 JSON 对象一文。
	space 可选:
		指定缩进用的空白字符串，用于美化输出（pretty-print）；
		如果参数是个数字，它代表有多少的空格；上限为10。该值若小于1，则意味着没有空格；
		如果该参数为字符串（当字符串长度超过10个字母，取其前10个字母），该字符串将被作为空格；
		如果该参数没有提供（或者为 null），将没有空格。
	返回值:
	一个表示给定值的JSON字符串。

``` javascript
const json_str = '{"result":true, "count":42}';
const json = { "name":"MYZ" };
console.log(typeof(json_str));	// > "string"
console.log(json_str);			// > "{"result":true, "count":42}"
console.log(typeof(json));		// > "object"
console.log(json);				// > Object { name: "MYZ" }

// JSON.stringify() 将值转换为相应的JSON格式, 也就是将JSON对象转换为字符串：
// JSON对象可以进行如json["name"]来取值，JSON字符串就行如‘{ "name":"MYZ" }’
json_ify = JSON.stringify(json);
console.log(typeof(json_ify));	// > "string"
console.log(json_ify);			// > "{"name":"MYZ"}"

json_parse = JSON.parse(json_ify);
console.log(typeof(json_parse));	// > "object"
console.log(json_parse);			// > Object { name: "MYZ" }

console.log(json["name"]);			// > "MYZ"
console.log(json_parse["name"]);	// > "MYZ"
console.log(json["name"] === json_parse["name"]);	// > true
```

``` javascript
const json_str = '{"result":true,"count":42}';
const json = { "name":"MYZ" };

var obj1 = JSON.parse(json_str);
var obj2 = JSON.stringify(obj1);
console.log(json_str);			> "{"result":true,"count":42}"
console.log(obj2);				> "{"result":true,"count":42}"
console.log(obj2 === json_str);	> true
```

``` javascript
const json_str = '{"result":true, "count":42}';	//字符串中间有一个空格
const json = { "name":"MYZ" };

var obj1 = JSON.parse(json_str);
var obj2 = JSON.stringify(obj1);	// 转化的字符串中间没有空格
console.log(json_str);			// > "{"result":true,"count":42}"
console.log(obj2);				// > "{"result":true,"count":42}"
console.log(obj2 === json_str);	// > false // 就因为多了空格因此两个字符串不相等
```

``` javascript
const json_str = '{"result": true,"count":42}';
const json = { "name":"MYZ" };

var obj1 = JSON.parse(json_str);
var obj2 = JSON.stringify(obj1);
console.log(json_str);			// > "{"result":true,"count":42}"
console.log(obj2);				// > "{"result":true,"count":42}"
console.log(obj2 === json_str);	// > false
```

### html+JS 实现循环编辑HTML标签
![](pic.JPG)

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Employee</title>
</head>
<body>
    <div class="znfz-cont">
        <ul id="data" class="znfz-list">
        </ul>
    </div>
    
    <div id="myDIV"> 
        <p class="child"> div 元素中 class="child" 的 p </p>
        <p class="child"> div 元素中 class="child" 的另外一个 p </p>
        <p>在 div 中的 p 元素插入 <span class="child">class="child" 的 span 元素</span> 。</p>
    </div>
    <p>点击按钮为 id="myDIV" 的 div 元素中所有 class="child" 元素添加背景颜色。 </p>
    <button onclick="myFunction()">点我</button>
    <p><strong>注意:</strong> Internet Explorer 8 及更早 IE 版本不支持 getElementsByClassName() 方法。</p>

    <script>
        function myFunction() {
            var x = document.getElementById("myDIV");
            var y = x.getElementsByClassName("child");
            var i;
            for (i = 0; i < y.length; i++) {
                y[i].style.backgroundColor = "red";
            }
        }
    </script>

</body>
<script>
var lists =[
    {
        "content":"尾矿库风险单元划分?",
        "read_num":256
    },
    {
        "content":"尾矿库今年发生的事故?",
        "read_num":65
    }
]
var str="";
for (var i = 0; i < lists.length; i++) {
    str += "<li class='znfz-num'>" + i + "</li>";
    str += "<router-link class='znfz-txt' to=''>" + lists[i].content + "</router-link>"
    str += "<span class='read-num'>" + lists[i].read_num + "</span>"
}

// 下面两种方法都可以，只不过getElementsByClassName返回的是数组元素，需要加上索引值如[0]
//document.getElementById("data").innerHTML=str;
document.getElementsByClassName("znfz-list")[0].innerHTML=str;

var name = "张三";
var age = 20;
console.log(`${name}的年龄是${age}`);

</script>
</html>
```






