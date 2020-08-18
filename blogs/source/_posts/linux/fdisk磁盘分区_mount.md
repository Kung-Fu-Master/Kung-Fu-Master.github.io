---
title: fdisk磁盘分区
tags: 
categories:
- linux
---

## **查看分区**
查看磁盘格式类型, UUID, 挂载点

	$ lsblk -f
	 NAME            FSTYPE      LABEL UUID                                   MOUNTPOINT
	 sda
	 ├─sda1          xfs               eadc654f-b75a-4d81-8e17-910031209006   /var/lib/kubelet/pods/0ef567bf-3636-455b-a585-6a9d8ab1b2dd/volumes/kubernetes.io~local-volume/local-pv-f
	 ├─sda2          xfs               47f36b35-c3fd-4374-86d6-0e56bb72eb02   /mnt/disks/47f36b35-c3fd-4374-86d6-0e56bb72eb02
	 ├─sda3          xfs               fea5b28a-8ac4-4736-9487-42517fd242de   /var/lib/kubelet/pods/e447c269-bbfd-4d59-9e01-432f14b75b26/volumes/kubernetes.io~local-volume/local-pv-3
	 ├─sda4          xfs               516c6d3c-c639-4092-82b6-0a82d09edff3   /var/lib/kubelet/pods/794961eb-3eb7-4cc2-b252-8d39d496b298/volumes/kubernetes.io~local-volume/local-pv-b
	 └─sda5          xfs               a7b1ca2d-5d08-411e-83e5-9fff59d554be   /mnt/disks/a7b1ca2d-5d08-411e-83e5-9fff59d554be
	 sdb             LVM2_member       OvpWBO-5AmD-7b4H-Plsn-twvB-k2oC-A5Aaca
	 └─ceph--7ee2d5aa--c451--44d1--8ff8--0d6394d5fb48-osd--data--9ca735f4--03c4--4f1d--89d8--bcd78778ce6c
	 
	 sdc             LVM2_member       3x1Vwv-POeH-igqu-yuRn-ex2l-oDp3-9FaFPi
	 └─ceph--c0d1ccc4--5130--4697--b9a2--e9c58cbc5f7b-osd--data--91d305f7--cfc5--4ca1--b787--35fefcc6f938
	 
	 nvme0n1
	 ├─nvme0n1p1     vfat              3E80-8EF9                              /boot/efi
	 ├─nvme0n1p2     xfs               d4e9a380-5394-47ff-8682-9f3ec689060c   /boot
	 └─nvme0n1p3     LVM2_member       2JVUdy-83cp-vH1n-LLAV-vMwZ-twiw-bhBNDH
	   ├─centos-root xfs               51083aeb-a167-4fe9-aced-fa14c2a18953   /
	   ├─centos-swap swap              ebcd1018-d259-4d2c-bfd0-46abdc143201
	   └─centos-home xfs               ada9d310-ffee-4b08-9350-3e222614c11b   /home
查看磁盘大小等信息

	$ lsblk
	 NAME                                    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
	 sda                                       8:0    0   1.8T  0 disk
	 ├─sda1                                    8:1    0    10G  0 part /var/lib/kubelet/pods/0ef567bf-3636-455b-a585-6a9d8ab1b2dd/volumes/kubernetes.io~local-volume/local-pv-f902c4d0
	 ├─sda2                                    8:2    0    10G  0 part /mnt/disks/47f36b35-c3fd-4374-86d6-0e56bb72eb02
	 ├─sda3                                    8:3    0    30G  0 part /var/lib/kubelet/pods/e447c269-bbfd-4d59-9e01-432f14b75b26/volumes/kubernetes.io~local-volume/local-pv-3f8c242c
	 ├─sda4                                    8:4    0   200G  0 part /var/lib/kubelet/pods/794961eb-3eb7-4cc2-b252-8d39d496b298/volumes/kubernetes.io~local-volume/local-pv-bcf24291
	 └─sda5                                    8:5    0   500G  0 part /mnt/disks/a7b1ca2d-5d08-411e-83e5-9fff59d554be
	 sdb                                       8:16   0   1.8T  0 disk
	 └─ceph--7ee2d5aa--c451--44d1--8ff8--0d6394d5fb48-osd--data--9ca735f4--03c4--4f1d--89d8--bcd78778ce6c
	                                         253:3    0   1.8T  0 lvm
	 sdc                                       8:32   0   1.8T  0 disk
	 └─ceph--c0d1ccc4--5130--4697--b9a2--e9c58cbc5f7b-osd--data--91d305f7--cfc5--4ca1--b787--35fefcc6f938
	                                         253:4    0   1.8T  0 lvm
	 nvme0n1                                 259:0    0   477G  0 disk
	 ├─nvme0n1p1                             259:1    0   200M  0 part /boot/efi
	 ├─nvme0n1p2                             259:2    0     1G  0 part /boot
	 └─nvme0n1p3                             259:3    0 475.8G  0 part
	   ├─centos-root                         253:0    0    50G  0 lvm  /
	   ├─centos-swap                         253:1    0     4G  0 lvm
	   └─centos-home                         253:2    0 421.8G  0 lvm  /home

## **执行分区**

	$ fdisk /dev/sda
	 WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.
	 Welcome to fdisk (util-linux 2.23.2).
	 
	 Changes will remain in memory only, until you decide to write them.
	 Be careful before using the write command.
	 
	 Command (m for help): `p`
	 
	 Disk /dev/sda: 1920.4 GB, 1920383410176 bytes, 3750748848 sectors
	 Units = sectors of 1 * 512 = 512 bytes
	 Sector size (logical/physical): 512 bytes / 4096 bytes
	 I/O size (minimum/optimal): 4096 bytes / 4096 bytes
	 Disk label type: gpt
	 Disk identifier: 6427BFAE-2D0B-4DBB-91E7-80D20C8A1DA7
	 
	 #         Start          End    Size  Type            Name
	  1         4096     20971519     10G  Microsoft basic primary
	  2     20971520     41943039     10G  Microsoft basic primary
	  3     41943040    104857599     30G  Microsoft basic primary
	  4    104857600    524287999    200G  Microsoft basic primary
	  5    524288000   1572863999    500G  Microsoft basic primary
	 
	 Command (m for help): `n`
	 Partition number (6-128, default 6):
	 First sector (34-3750748814, default 1572864000):
	 Last sector, +sectors or +size{K,M,G,T,P} (1572864000-3750748814, default 3750748814): `+10G`
	 Created partition 6
	 
	 Command (m for help): `p`
	 
	 Disk /dev/sda: 1920.4 GB, 1920383410176 bytes, 3750748848 sectors
	 Units = sectors of 1 * 512 = 512 bytes
	 Sector size (logical/physical): 512 bytes / 4096 bytes
	 I/O size (minimum/optimal): 4096 bytes / 4096 bytes
	 Disk label type: gpt
	 Disk identifier: 6427BFAE-2D0B-4DBB-91E7-80D20C8A1DA7
	 
	 #         Start          End    Size  Type            Name
	  1         4096     20971519     10G  Microsoft basic primary
	  2     20971520     41943039     10G  Microsoft basic primary
	  3     41943040    104857599     30G  Microsoft basic primary
	  4    104857600    524287999    200G  Microsoft basic primary
	  5    524288000   1572863999    500G  Microsoft basic primary
	  6   1572864000   1593835519     10G  Linux filesyste
	 
	 Command (m for help): `w`
	 The partition table has been altered!
	 
	 Calling ioctl() to re-read partition table.
	 
	 WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
	 The kernel still uses the old table. The new table will be used at
	 the next reboot or after you run partprobe(8) or kpartx(8)
	 Syncing disks.

## **不重启机器,Kernel重新加载磁盘table**
查看磁盘文件

	$ ls /dev/sda*
	 /dev/sda  /dev/sda1  /dev/sda2  /dev/sda3  /dev/sda4  /dev/sda5 	// 没有sda6磁盘
进行Kernel sync disks

	$ partprobe /dev/sda
	$ lsblk -f
	 NAME            FSTYPE      LABEL UUID                                   MOUNTPOINT
	 sda
	 ├─sda1          xfs               eadc654f-b75a-4d81-8e17-910031209006   /var/lib/kubelet/pods/0ef567bf-3636-455b-a585-6a9d8ab1b2dd/volumes/kubernetes.io~local-volume/local-pv-f
	 ......
	 └─sda6
	 sdb             LVM2_member       OvpWBO-5AmD-7b4H-Plsn-twvB-k2oC-A5Aaca
	 └─ceph--7ee2d5aa--c451--44d1--8ff8--0d6394d5fb48-osd--data--9ca735f4--03c4--4f1d--89d8--bcd78778ce6c
	 
	 sdc             LVM2_member       3x1Vwv-POeH-igqu-yuRn-ex2l-oDp3-9FaFPi
	 └─ceph--c0d1ccc4--5130--4697--b9a2--e9c58cbc5f7b-osd--data--91d305f7--cfc5--4ca1--b787--35fefcc6f938
	 ......

## **删除分区**

	$ fdisk /dev/sda
	 m 		// 查看命令
	 d 		// 删除分区
	 6 		// 选择要删除的partition
	 w 		// 输入 w  保存，这个时候分区以及删除了

## **格式化分区**
在设备上格式化成指定格式的文件系统；  centos 7以后的版本默认使用xfs格式  ；也可以指定 ext3\4格式

	fs：指定建立文件系统时的参数；
	-t<文件系统类型>：指定要建立何种文件系统；
	-v：显示版本信息与详细的使用方法；
	-V：显示简要的使用方法；
	-c：在制做档案系统前，检查该partition是否有坏轨。
格式为xfs,所以使用mkfs.xfs命令。**`如果已有其他文件系统创建在此分区，必须加上"-f"参数来覆盖它`**

	mkfs.xfs -f  -i size=512 -l size=128m,lazy-count=1 -d agcount=64 /dev/xvda3
	-i size=512 : 默认的值是256KB，当内容小于这个值时，写到inode中，超过这个值时，写到block中。
	-l size=128m :默认值的是10m，修改这个参数成128m，可以显著的提高xfs文件系统删除文件的速度，当然还有其它，如拷贝文件的速度。
	-l lazy-count=1: 值可以是0或者1；默认值是0;在一些配置上显著提高性能；
	-d agcount=4 : 默认值是根据容量自动设置的。可以设置成1/2/4/16等等，这个参数可以调节对CPU的占用率，值越小，占用率越低；因为我的硬盘为2T的大硬盘，所以设置64；
**Use Case:**

	$ mkfs.xfs -f /dev/sda6        格式化sda6磁盘
	 mkfs.xfs -f /dev/sda6
	 meta-data=/dev/sda6              isize=512    agcount=4, agsize=655360 blks
	          =                       sectsz=4096  attr=2, projid32bit=1
	          =                       crc=1        finobt=0, sparse=0
	 data     =                       bsize=4096   blocks=2621440, imaxpct=25
	          =                       sunit=0      swidth=0 blks
	 naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
	 log      =internal log           bsize=4096   blocks=2560, version=2
	          =                       sectsz=4096  sunit=1 blks, lazy-count=1
	 realtime =none                   extsz=4096   blocks=0, rtextents=0
	$ lsblk
	 NAME                                    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
	 sda                                       8:0    0   1.8T  0 disk
	 ├─sda1                                    8:1    0    10G  0 part /var/lib/kubelet/pods/0ef567bf-3636-455b-a585-6a9d8ab1b2dd/volumes/kubernetes.io~local-volume/local-pv-f902c4d0
	 ......
	 └─sda6                                    8:6    0    10G  0 part
	$ lsblk -f
	 NAME            FSTYPE      LABEL UUID                                   MOUNTPOINT
	 sda
	 ├─sda1          xfs               eadc654f-b75a-4d81-8e17-910031209006   /var/lib/kubelet/pods/0ef567bf-3636-455b-a585-6a9d8ab1b2dd/volumes/kubernetes.io~local-volume/local-pv-f
	 ......
	 └─sda6          xfs               8c653ffc-4b28-47d1-9e7f-32f8d969757a

## **挂载分区**

	$ mkdir /d1 
	$ mount /dev/sda6 /d1 
**设置开机自动挂载新建分区**

	$ vim /etc/fstab
	 #
	 # /etc/fstab
	 # Created by anaconda on Fri Apr  3 16:29:28 2020
	 #
	 # Accessible filesystems, by reference, are maintained under '/dev/disk'
	 # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
	 #
	 /dev/mapper/centos-root /                       xfs     defaults        0 0
	 /dev/mapper/centos-home /home                   xfs     defaults        0 0
	 UUID=eadc654f-b75a-4d81-8e17-910031209006 /mnt/disks/eadc654f-b75a-4d81-8e17-910031209006 xfs defaults 0 2
	 UUID=47f36b35-c3fd-4374-86d6-0e56bb72eb02 /mnt/disks/47f36b35-c3fd-4374-86d6-0e56bb72eb02 xfs defaults 0 2
	 UUID=fea5b28a-8ac4-4736-9487-42517fd242de /mnt/disks/fea5b28a-8ac4-4736-9487-42517fd242de xfs defaults 0 2
	 UUID=516c6d3c-c639-4092-82b6-0a82d09edff3 /mnt/disks/516c6d3c-c639-4092-82b6-0a82d09edff3 xfs defaults 0 2
	 UUID=a7b1ca2d-5d08-411e-83e5-9fff59d554be /mnt/disks/a7b1ca2d-5d08-411e-83e5-9fff59d554be xfs defaults 0 2
	 UUID=8c653ffc-4b28-47d1-9e7f-32f8d969757a /mnt/disks/My-Directory xfs default 0 2
/etc/fstab文件负责配置Linux开机时自动挂载的分区

	 第一列可以是实际分区名，也可以是实际分区的卷标（Lable）
	 第二列是挂载点,挂载点必须为当前已经存在的目录
	 第三列为此分区的文件系统类型
	 第四列是挂载的选项，用于设置挂载的参数
	 * auto: 系统自动挂载，fstab默认就是这个选项
	 * defaults: rw, suid, dev, exec, auto, nouser, and async.
	 * noauto 开机不自动挂载
	 * nouser 只有超级用户可以挂载
	 * ro 按只读权限挂载
	 * rw 按可读可写权限挂载
	 * user 任何用户都可以挂载
	 请注意光驱和软驱只有在装有介质时才可以进行挂载，因此它是noauto
	 第五列是dump备份设置,当其值设置为1时，将允许dump备份程序备份；设置为0时，忽略备份操作；
	 第六列是fsck磁盘检查设置,其值是一个顺序。当其值为0时，永远不检查；而 / 根目录分区永远都为1。其它分区从2开始，数字越小越先检查，如果两个分区的数字相同，则同时检查。

## **卸载分区**

	$ umount /dev/sda6




