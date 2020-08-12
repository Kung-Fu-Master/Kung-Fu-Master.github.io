---
title: 查看SSD
tags:
categories:
- linux
---
[root@wlp10 ~]# lsscsi -ls
[6:0:0:0]    disk    ATA      INTEL SSDSC2BB48 0370  /dev/sda    480GB
  state=running queue_depth=32 scsi_level=6 type=0 device_blocked=0 timeout=30
[7:0:0:0]    disk    ATA      INTEL SSDSC2BB48 0101  /dev/sdb    480GB
  state=running queue_depth=32 scsi_level=6 type=0 device_blocked=0 timeout=30
[8:0:0:0]    disk    ATA      INTEL SSDSC2BB48 0370  /dev/sdc    480GB
  state=running queue_depth=32 scsi_level=6 type=0 device_blocked=0 timeout=30
[9:0:0:0]    disk    ATA      INTEL SSDSC2BG01 0015  /dev/sdd   1.20TB
  state=running queue_depth=32 scsi_level=6 type=0 device_blocked=0 timeout=30
[root@wlp10 ~]#
看第四列就知道是否是SSD硬盘了


[root@wlp10 ~]# lsscsi -h
Usage: lsscsi   [--classic] [--device] [--generic] [--help] [--hosts]
                [--kname] [--list] [--lunhex] [--long] [--protection]
                [--scsi_id] [--size] [--sysfsroot=PATH] [--transport]
                [--verbose] [--version] [--wwn] [<h:c:t:l>]
  where:
    --classic|-c      alternate output similar to 'cat /proc/scsi/scsi'
    --device|-d       show device node's major + minor numbers
    --generic|-g      show scsi generic device name
    --help|-h         this usage information
    --hosts|-H        lists scsi hosts rather than scsi devices
    --kname|-k        show kernel name instead of device node name
    --list|-L         additional information output one
                      attribute=value per line
    --long|-l         additional information output
    --lunhex|-x       show LUN part of tuple as hex number in T10 format;
                      use twice to get full 16 digit hexadecimal LUN
    --protection|-p   show target and initiator protection information
    --protmode|-P     show negotiated protection information mode
    --scsi_id|-i      show udev derived /dev/disk/by-id/scsi* entry
    --size|-s         show disk size
    --sysfsroot=PATH|-y PATH    set sysfs mount point to PATH (def: /sys)
    --transport|-t    transport information for target or, if '--hosts'
                      given, for initiator
    --verbose|-v      output path names where data is found
    --version|-V      output version string and exit
    --wwn|-w          output WWN for disks (from /dev/disk/by-id/wwn*)
    <h:c:t:l>         filter output list (def: '*:*:*:*' (all))

List SCSI devices or hosts, optionally with additional information
[root@wlp10 ~]#

From <https://www.cnblogs.com/laozhuang/p/7110438.html> 
