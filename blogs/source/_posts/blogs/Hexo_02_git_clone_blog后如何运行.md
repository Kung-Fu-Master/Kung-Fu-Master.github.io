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

## **git设置忽略文件(夹)和目录**
博客clone下来后执行`hexo clean; hexo g;`后, 再查看git状态`git status`, 会看到blogs/public/* 和 db.json处于modified状态，如果想每次生成博客后忽略 blogs/public/等下的文件修改情况, 参考以下方式.  
1. gitbash 命令进入本地git库目录
2. 博客根目录创建.gitignore文件
3. 修改文件, 添加如下的忽略正则内容


	$ vim .gitignore
	.idea			// 忽略.idea文件夹及文件夹下的文件的修改
	*.html			// 忽略以.html结尾的文件的修改
	blogs/public/	// 忽略blogs/public/下的所有文件的修改
	db.json			// 忽略db.json文件的修改
	blogs/.deploy_git/*

**Note:** .gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理也就是已经执行了`git add <文件(夹)A>;`，则修改.gitignore, 忽略`文件(夹)A`是无效的。  
**正确的做法**是在clone下来的仓库中手动执行 `git rm <文件(夹)A>;`如`git rm blogs/public/*`,然后再`git commit "*"; git push origin master;`, 远程仓库的此文件(夹)也删除后，再在本地生成的`文件(夹)A`, git就可以忽略它的修改.  
