---
title: git commands
tags: git
categories:
- technologies
- git
---

## **git 放弃本地修改**
**1. 未使用 git add 缓存代码时**

	$ git checkout -- <filePathName> // git checkout -- README.md
但是此命令不会删除掉刚新建的文件。因为刚新建的文件还没已有加入到 git 的管理系统中。所以对于git是未知的。自己手动删除就好了.
**2. 已经使用了  git add 缓存了代码**

	$ git reset HEAD <filePathName> // git reset HEAD readme.md
	放弃所以的缓存可以使用如下命令:
	$ git reset HEAD .
此命令用来清除 git  对于文件修改的缓存。相当于撤销 git add 命令所在的工作。在使用本命令后，本地的修改并不会消失，而是回到了如（1）所示的状态。继续用（1）中的操作，就可以放弃本地的修改.
**3. 已经用 git commit  提交了代码**

	$ git reset --hard HEAD^	// 回退到上一次commit的状态
	回退到任意版本
	$ git log			// 查看git的提交历史
	$ git reset --hard <commitid>	//

## **git 创建新的分支**
1.使用git bash 进入到已有项目根目录下，执行如下命令创建分支


	$git checkout -b dev-01	// 此命令相当于`git branch dev-01; git checkout dev-01;`
2.查看当前分支


	$ git branch
它就会有如下显示：
	* dev-01
	  master

3.将新建分支提交到远程仓库, 远程会自动生成同名新分支


	$ git push origin dev-01
4.拉取远程分支，但会发现提示没有指定要与哪个分支合并，无法与远程仓库进行关联，所以需要先关联，后拉取


	$ git branch --set-upstream-to=origin/dev-01
	$ git pull
查看关联情况

	$ git branch -vv
5. 最后把本地代码推上去


	$ git add *
	$ git commit -m 'your commit info'
	$ git push origin dev-01
6. 切换分支执行上面命令后, 查看远端github仓库新分支仍然有源分支文件, 可以:


	$ git rm <源文件(夹)>
	$ git commit -m "*"
	$ git push origin dev-01

## **git 切换分支**
1. 查看远程分支


	$ git branch -a
	* master
	remotes/origin/HEAD -> origin/master
	remotes/origin/v0.9rc1

2. 查看本地分支


	$ git branch
	* master

3. 切换分支


	$ git checkout -b v0.9rc1 origin/v0.9rc1	//如果本地已经有v0.9rc1分支了就可以直接`git checkout v0.9rc1`
	Branch v0.9rc1 set up to track remote branch v0.9rc1 from origin.
	Switched to a new branch 'v0.9rc1'
已经切换到v0.9rc1分支了

	$ git branch
	master
	* v0.9rc1

切换回master分支

	$ git checkout master
	Switched to branch 'master'
	Your branch is up-to-date with 'origin/master'.

## **分支的新建与合并**
[https://git-scm.com/book/zh/v2/Git-分支-分支的新建与合并](https://git-scm.com/book/zh/v2/Git-分支-分支的新建与合并)


## **git diff 加上颜色**

	$ git config color.ui true

## **git reset到指定commit log**

	$ git log
	$ git reset --hard <Commit-ID>


