---
title: curl & wget
tags: 
categories:
- linux
---

## Linux curl命令详解
在Linux中curl是一个利用URL规则在命令行下工作的文件传输工具，可以说是一款很强大的http命令行工具。它支持文件的上传和下载，是综合传输工具，但按传统，习惯称url为下载工具。
常见参数：
	-A/--user-agent <string>              设置用户代理发送给服务器
	-b/--cookie <name=string/file>    cookie字符串或文件读取位置
	-c/--cookie-jar <file>                    操作结束后把cookie写入到这个文件中
	-C/--continue-at <offset>            断点续转
	-D/--dump-header <file>              把header信息写入到该文件中
	-e/--referer                                  来源网址
	-f/--fail                                          连接失败时不显示http错误
	-o/--output                                  把输出写到该文件中
	-O/--remote-name                      把输出写到该文件中，保留远程文件的文件名
	-r/--range <range>                      检索来自HTTP/1.1或FTP服务器字节范围
	-s/--silent                                    静音模式。不输出任何东西
	-T/--upload-file <file>                  上传文件
	-u/--user <user[:password]>      设置服务器的用户和密码
	-w/--write-out [format]                什么输出完成后
	-x/--proxy <host[:port]>              在给定的端口上使用HTTP代理
	-#/--progress-bar                        进度条显示当前的传送状态
wget更像一个迅雷，是个专门用来下载文件的下载利器.

### 基本命令
	curl -O http://man.linuxde.net/text.iso                    #O大写，不用O只是打印内容不会下载
	wget http://www.linuxde.net/text.iso                       #不用参数，直接下载文件
	root@alpha:/home/test# curl -O http://man.linuxde.net/text.iso
	% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
									Dload  Upload   Total   Spent    Left  Speed
	100  6332  100  6332    0     0   441k      0 --:--:-- --:--:-- --:--:--  441k
	root@alpha:/home/test#

### 保存网页
	$ curl http://www.linux.com >> linux.html

### 下载文件
	$ curl -O http://man.linuxde.net/text.iso                    #O大写，不用O只是打印内容不会下载
	$ wget http://www.linuxde.net/text.iso                       #不用参数，直接下载文件

### 下载文件并重命名
	$ curl -o rename.iso http://man.linuxde.net/text.iso         #o小写
	$ wget -O rename.zip http://www.linuxde.net/text.iso         #O大写

### 断点续传
在windows中，我们可以使用迅雷这样的软件进行断点续传。curl可以通过内置option:-C同样可以达到相同的效果
如果在下载dodo1.JPG的过程中突然掉线了，可以使用以下的方式续传
	curl -O -C -URL http://man.linuxde.net/text.iso            #C大写
	wget -c http://www.linuxde.net/text.iso                    #c小写

### 限速下载
	$ curl --limit-rate 50k -O http://man.linuxde.net/text.iso
	$ wget --limit-rate=50k http://www.linuxde.net/text.iso

### 显示响应头部信息
	$ curl -I http://man.linuxde.net/text.iso
	$ wget --server-response http://www.linuxde.net/test.iso

### wget利器--打包下载网站
	$ wget --mirror -p --convert-links -P /var/www/html http://man.linuxde.net/

### -O(大写)保存网页中的文件, 要注意这里后面的url要具体到某个文件，不然抓不下来
	$ curl -O http://www.linux.com/hello.sh

### -x来支持设置代理
	$ curl -x 192.168.100.100:1080 http://www.linux.com	

### 保存http的response里面的cookie信息。内置option:-c（小写）
	$ curl -c cookiec.txt  http://www.linux.com

### 保存http的response里面的header信息。内置option: -D
	$ curl -D cookied.txt http://www.linux.com

### 很多网站都是通过监视你的cookie信息来判断你是否按规矩访问他们的网站的，因此我们需要使用保存的cookie信息。内置option: -b
	$ curl -b cookiec.txt http://www.linux.com


### 以服务器上的名称保存文件到本地: -O（大写)
	$ curl -O http://www.linux.com/dodo1.JPG

### 循环下载
	$ curl -O http://www.linux.com/dodo[1-5].JPG
	$ curl -O http://www.linux.com/{hello,bb}/dodo[1-5].JPG
	下载http://www.linux.com/hello/dodo[1-5].JPG 和 http://www.linux.com/bb/dodo[1-5].JPG 文件

### 显示进度条
	$ curl -# -O http://www.linux.com/dodo1.JPG
	root@alpha:/home/zhan/test# curl -# -O http://www.linux.com/dodo1.JPG
	####################################################################################### 100.0%


### 上传文件
	$ curl -T dodo1.JPG -u 用户名:密码 ftp://www.linux.com/img/

### Additional

	root@alpha:/home/zhan/images_test# curl 10.239.140.186:8081
	You have hit fef0dd414fe4
	root@alpha:/home/zhan/images_test# wget 10.239.140.186:8081
	--2020-05-11 14:59:53--  http://10.239.140.186:8081/
	Resolving child-prc.intel.com (child-prc.intel.com)... 10.239.4.100
	Connecting to child-prc.intel.com (child-prc.intel.com)|10.239.4.100|:913... connected.
	Proxy request sent, awaiting response... 200 OK
	Length: unspecified
	Saving to: ‘index.html’
	
	index.html                              [ <=>                                                                ]      24  --.-KB/s    in 0s
	
	2020-05-11 14:59:53 (3.27 MB/s) - ‘index.html’ saved [24]
	
	root@alpha:/home/zhan/images_test# ls
	index.html
	root@alpha:/home/zhan/images_test# cat index.html
	You have hit fef0dd414fe4
	root@alpha:/home/zhan/images_test#









