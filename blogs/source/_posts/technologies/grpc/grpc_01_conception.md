---
title: grpc 01 conception
tags: 
categories:
- technologies
- grpc
---

## **Conception**
所谓RPC(remote procedure call 远程过程调用)框架实际是提供了一套机制，使得应用程序之间可以进行通信，而且也遵从server/client模型。使用的时候客户端调用server端提供的接口就像是调用本地的函数一样。如下图所示就是一个典型的RPC结构图
![](1.png)
gRPC其实就是google在rpc基础上定义了一套自己的使用方式, 也可说成是google的prc方式开发.  
学习gPRC得学习Protocol Buffers, 它可以用来定义消息和服务.  
然后只需要实现服务即可, 剩余的gPRC代码将会自动为你生成.  
.proto这个文件可以使用于十几种开发语言(包括服务端和客户端), 并且允许你使用同一个框架来支持每秒百万级以上的PRC调用.  

### 开发模式
gPRC使用合约优先的API开发模式, 默认使用Protocol buffers(protobuf)作为接口设计语言(IDL), 这个.proto文件包括两部分:
 * gPRC服务的定义
 * 服务端和客户端之间传递的消息

## **开发环境**
VSCode 扩展插件: vscode-proto3, Clang-Format
Windows还需要安装Clang, Widnows安装地址：[Clang download for windows](http://www.llvm.org/releases/download.html) Windows (64-bit) (.sig)

下载后安装过程中可以选择Add LLVM to the system PATH for all users

## VSCode 一些常用操作
Ctrl+, 打开设置面板
Ctrl+shift+p 搜索theme主题样式 -> File icon theme -> Install Additional File Icon Themes... -> 可以选择Material Icon Theme， 装完主题之后点击右下角蹦出的Activate激活  





