---
title: EOF
tags: 
categories:
- linux
---

## EOF
通过cat配合重定向能够生成文件并追加操作,在它之前先熟悉几个特殊符号:

```
	< :输入重定向
	> :输出重定向
	>> :输出重定向,进行追加,不会覆盖之前内容
	<< :标准输入来自命令行的一对分隔号的中间内容.
```

"<< EOF EOF" 的作用是在命令执行过程中用户自定义输入，它类似于起到一个临时文件的作用，只是比使用文件更方便灵活

EOF是END Of File的缩写,表示自定义终止符.既然自定义,那么EOF就不是固定的,可以随意设置别名,在linux按ctrl-d就代表EOF.
EOF一般会配合cat能够多行文本输出.
其用法如下:

```
	<<EOF        //开始
	....
	EOF            //结束
```

还可以自定义，比如自定义：

```
	<<BBB        //开始
	....
	BBB              //结束
```

1. 向文件test.sh里输入内容。 

```shell
	$ cat << EOF > test.sh 或者 $ cat > test.sh << EOF
	> 123123123
	> 3452354345
	> asdfasdfs
	> EOF
```

追加内容

```shell
	$ cat << EOF >>test.sh 或者 $ cat >> test.sh << EOF
	> 7777
	> 8888
	> EOF
```

覆盖

```shell
	$ cat << EOF >test.sh 或者 $ cat > test.sh << EOF
	> 55555
	> EOF
```

2. 自定义EOF，比如自定义为wang

```shell
	$ cat << wang > haha.txt 或者 $ cat > test.sh << wang
	> ggggggg
	> 6666666
	> wang
```




