---
title: Hexo 02 git clone blog后如何运行
tags:
categories:
- blogs
---

## **1. 确保系统已安装node.js, git工具**

## **2. cmd管理员身份进入git clone下来的文件夹**

	$ cd ...\blogs_backup\blogs
	$ npm install hexo-deployer-git --save
## **3. 生成部署博客**

	$ hexo g    //generate，md生成html
	$ hexo s    //start，开启服务, 网页上输入http://localhost:4000 即可查看
添加***.md文件后

	#hexo d        //deploy部署，中途需要输入账号密码, 稍等一会即可chrome网页上输入https://Kung-Fu-Master.github.io 查看
但是此时新添加的.md文件没有上传到blogs_backup git仓库中，需要打开git bash, git add这个.md文件再git commit -m "add new .md", git push, 完成blogs_backup仓库的更新.

## **免密 $hexo d 方法**
### **第一种, 亲测**
git bash终端 cd 到博客副本文件夹, 输入: $git config --global credential.helper store  
接下来 $git push 需要输入密码，以后 $git push 就不用了, 然后 $hexo d 也就不需要了  

原因是执行 $git config --global credential.helper store 后会创建文件 C:\Users\hp\.git-credentials(Windows系统下)  
里面内容: https://git账户名:登陆git密码@github.com  
### **第二种, 网上**
创建文件 C:\Users\hp\.git-credentials 并添加内容 https://{username}:{password}@github.com
添加git config内容

	$git config --global credential.helper store
执行此命令后，用户主目录下(C:\Users\hp\)的.gitconfig文件会多了一项：[credential]

	helper = store
重新git push就不需要用户名密码了

