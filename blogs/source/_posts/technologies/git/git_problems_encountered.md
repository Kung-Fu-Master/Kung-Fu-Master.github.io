---
title: git problems encountered
tags: git
categories:
- technologies
- git
---

## 查看github 贡献最多和增长最快链接:
https://octoverse.github.com/#fastest-growing-oss-projects-by-contributors

## git branch只能看到master分支
执行`git init`命令后再执行`git clone ...`, 之后执行`git branch -a`, 发现只有master分支, 远程还有其它分支但是看不到.  

解决方法:

```xml
	$ vim .git/config
	// 添加如下内容
	[remote "origin"]
		url = https://***.git	// 跟git clone 后面跟的地址一样, 也就是项目所在地址.
		fetch = +refs/heads/*:refs/remotes/origin/*		// 必须加上这一行
```

之后回到上一层目录, 然后fetch 远程分支

``` shell
	$ git fetch origin
	输入用户名密码
	$ git branch -a
	即可查看到远程所有分支
```

## 解决git bash 终端显示中文乱码

在git bash的界面中右击空白处，弹出菜单，选择选项->文本->本地Locale，设置为zh_CN，而旁边的字符集选框选为UTF-8

上面方法在git add带中文文件时候会出现 "add不到匹配的文件" 或者 重启还是出现中文乱码,解决方法如下(亲测有效)：

1. 选择选项->文本->本地Locale，设置为(Defalult)
2. `git config --global core.quotepath false` 再重启gitbash终端可解决

## git push免密登陆方法

1. 创建文件 C:\Users\hp\.git-credentials(Windows系统, 惠普电脑, 其它种类OS和厂商路径类似)

打开并添加内容 https://{username}:{password}@github.com

2.添加git config内容

``` shell
$git config --global credential.helper store
```

执行此命令后，用户主目录下的.gitconfig文件会多了一项：[credential]
helper = store

重新git push就不需要用户名密码了

## git提示“warning: LF will be replaced by CRLF”
遇到此问题场景: 在windows上用git提交linux文件, 这是因为在文本处理中，CR（CarriageReturn），LF（LineFeed），CR/LF是不同操作系统上使用的换行符.

 * Dos和Windows平台: 使用回车（CR）和换行（LF）两个字符来结束一行，回车+换行(CR+LF)，即“\r\n”；
 * Mac 和 Linux平台: 只使用换行（LF）一个字符来结束一行，即“\n”；
所以我们平时在windows上编写文件的`回车符`应该确切来说叫做`回车换行符`.

许多 Windows 上的编辑器会悄悄把行尾的换行（LF）字符转换成回车（CR）和换行（LF），或在用户按下 Enter 键时，插入回车（CR）和换行（LF）两个字符.  

**影响：**
 * Unix/Mac系统下的文件在Windows里打开的话，所有文字会变成一行.  
 * 而Windows里的文件在Unix/Mac下打开的话，在每行的结尾可能会多出一个^M符号.  
 * Linux保存的文件在windows上用记事本看的话会出现黑点.  
通过一定方式进行转换统一:

```
	在linux下，命令unix2dos 是把linux文件格式转换成windows文件格式
	命令dos2unix 是把windows格式转换成linux文件格式.
```

**情况一:**
Git 可以在你提交时自动地把回车(CR)和换行(LF)转换成换行(LF), 而在检出代码时把换行(LF)转换成回车(CR)和换行(LF).  
如果是在 Windows 系统上，把它设置成 true，这样在检出代码时，换行会被转换成回车和换行.  

**提交时转换为LF，检出时转换为CRLF**

``` shell
	$ git config --global core.autocrlf true
```

**情况二:**
可以把 core.autocrlf 设置成 input 来告诉 Git 在提交时把回车和换行转换成换行，检出时不转换, 这样在 Windows 上的检出文件中会保留回车和换行，而在 Mac 和 Linux 上，以及版本库中会保留换行.  

**提交时转换为LF，检出时不转换**

``` shell
	$ git config --global core.autocrlf input
```

**情况三:**
如果你是 Windows 程序员，且正在开发仅运行在 Windows 上的项目，可以设置 false 取消此功能，把回车保留在版本库中.  

**提交检出均不转换**

``` shell
	$ git config --global core.autocrlf false
```

**你也可以在文件提交时进行safecrlf检查**

**拒绝提交包含混合换行符的文件**

``` shell
	git config --global core.safecrlf true   
```

**允许提交包含混合换行符的文件**

``` shell
	git config --global core.safecrlf false   
```

**提交包含混合换行符的文件时给出警告**

``` shell
	git config --global core.safecrlf warn
```

**通俗解释**
 * git 的 Windows 客户端基本都会默认设置 core.autocrlf=true，设置core.autocrlf=true, 只要保持工作区都是纯 CRLF 文件，编辑器用 CRLF 换行，就不会出现警告了.  
 * Linux 最好不要设置 core.autocrlf，因为这个配置算是为 Windows 平台定制.  
 * Windows 上设置 core.autocrlf=false，仓库里也没有配置 .gitattributes，很容易引入 CRLF 或者混合换行符（Mixed Line Endings，一个文件里既有 LF 又有CRLF）到版本库，这样就可能产生各种奇怪的问题.  





