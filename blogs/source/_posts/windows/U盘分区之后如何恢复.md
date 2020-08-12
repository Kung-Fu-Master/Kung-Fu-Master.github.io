---
title: U盘分区之后如何恢复
categories:
- windows
---

1. 插入U盘。
2. 按windows键，搜索cmd 选择以管理员身份运行。
3. 输入diskpart,按enter。
![](1.png)
4. 输入list disk,按enter。
![](2.png)
5. 之后会看到
disk 0
disk 1
如果你给你电脑磁盘分过区的话可能还有disk 2  、  disk 3等
![](3.png)
6. 输入select disk X(X代表磁盘后面的数字0、1，可磁盘的大小来判断数字是多少，一般是1),按enter
![](4.png)
7. 输入clean，按enter
![](5.png)
8. 
以上完成之后，到“磁盘管理”选择U盘，新建简单卷，并重命名你的U盘

打开“磁盘管理”方法:
任务栏"Search"输入"disk management", 选择运行"Create and format hard disk partitions"
"新建简单卷"方法:
打开磁盘管理后，选中U盘所在Disk 如Disk1， 右击鼠标，选中第一个新建卷，默认下一步，之后需要格式化, "Volume lable"输入盘volume名字如OS就可以了.

