## blogs_backup
hexo部署博客最初始原文件

## hexo-theme-snippet主题

## 常见问题(FAQ)
参考官网: https://github.com/shenliyang/hexo-theme-snippet/blob/master/README.md

1. 搜索功能不能用，content.json文件找不到

需要安装hexo-generator-json-content插件, 到博客根目录执行如下命令安装.


	$ npm i hexo-generator-json-content@2.2.0 -S


## hexo-theme-stun主题
参考: https://theme-stun.github.io/docs/zh-CN/guide/primary.html

	$ npm install --save hexo-renderer-pug

## 分类页、标签页。默认没有启用。如果想启用它们

	//启用分类页，执行这条指令
	$ hexo new page categories
	
	// 启用标签页，执行这条指令
	$ hexo new page tags

修改 Front-Matter

在 Hexo 根目录下，找到 source/categories 或 source/tags 文件夹中的 Markdown 文件，设置 Front-Matter：

	---
	// 如果是分类页，设置这个
	type: "categories"
	
	// 如果是标签页，设置这个
	type: "tags"
	---

### 添加自定义按钮
1. 修改主题stun配置文件 _config.yml


	menu:
	  home: / || fas fa-home
	  article: javascript:; || fas fa-feather-alt

2. 找到 stun/languages 目录下的语言文件，根据你网站使用的语言选择对应的语言文件，例如：

zh-CN.yml：


	# 导航栏菜单
	menu:
	  ......
	  article: 文章

这样就添加好了自定义页面。


### 安装搜索插件

	$ npm install hexo-generator-search --save
配置插件

找到 Hexo 根目录下的 _config.yml 文件，添加以下字段：

	search:
	  path: search.json
	  field: post
	  content: true
有关插件的详尽信息和上述参数的含义，请查看插件的文档.
修改stun主题目录下的 _config.yml文件, 启用local search

	local_search:
	  # 是否启用
	  enable: true
生成数据

	$ hexo g
这样会在你网站根目录下的 public 的文件夹中，生成 search.json 文件，Stun 主题的本地搜索功能就是利用这个文件里的数据实现的。

	$ hexo clean
	$ hexo g
	$ hexo s
### 启用文章字数统计:

	$ npm install hexo-wordcount --save
然后重启 Hexo 服务器

### Font Awesome 字体图标对照表
reference: https://getkit.cn/resources/font-awesome/

### 文章置顶

想要使用文章置顶功能，首先你需要安装 Hexo 插件 hexo-generator-index-pin-top (opens new window)，然后执行命令：

	$ npm uninstall hexo-generator-index --save
	$ npm install hexo-generator-index-pin-top --save
最后，在文章的 Front-Matter 中，使用 top: true 来实现置顶。

设置文章置顶后，在文章列表中可以看到表示置顶的图标。







