---
title: U盘制作启动盘分区后恢复
tags: 
categories:
- linux
---

## **第一种操作步骤**
1. 插入U盘。
2. 按windows键，右键点击运行，再左键点击以管理员身份运行。
3. 输入diskpart,按enter。
![](diskpart.png)
4. 输入list disk,按enter。
![](list_disk.png)
5. 之后会看到
disk 0
disk 1
如果你给你电脑磁盘分过区的话可能还有disk 2  、  disk 3等
![](list_disk.png)
6. 输入select disk X(X代表磁盘后面的数字0、1，可磁盘的大小来判断数字是多少，一般是1),按enter
![](select_disk.png)
7. 输入clean，按enter
![](clean.png)
8. 
以上完成之后，到“磁盘管理”选择U盘，新建简单卷，并重命名你的U盘

打开“磁盘管理”方法:
任务栏"Search"输入"disk management", 选择运行"Create and format hard disk partitions"
"新建简单卷"方法:
打开磁盘管理后，选中U盘所在Disk 如Disk1， 右击鼠标，选中第一个新建卷，默认下一步，期间"Volume lable"输入盘volume名字就可以了.

## **第二种使用DiskGenius软件**
![](1.PNG)
![](2.PNG)

