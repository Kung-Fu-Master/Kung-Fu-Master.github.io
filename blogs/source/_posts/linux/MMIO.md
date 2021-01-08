---
title: MMIO
tags: 
categories:
- linux
---
Linux kernel目录
/usr/src/kernels/3.10.0-693.el7.x86_64/.config
/usr/src/kernels/3.10.0-957.21.3.el7.x86_64/.config
上面文件夹内基本都只有头文件，缺少kernel的源码文件
yumdownloader --source kernel  安装新的kernel源码

/dev/mem是linux下的一个字符设备, 源文件是kernel/drivers/char/mem.c, 这个设备文件是专门用来读写物理地址用的。里面的内容是所有物理内存的地址以及内容信息。通常只有root用户对其有读写权限。

```c
#include<sys/mman.h>
void *mmap(void *start, size_t length, int prot, int flags,
           int fd, off_t offset);
int munmap(void *start, size_t length);
```

mmap详细用法不在此展开, 特别注意参数start(一般赋值为NULL)和offset是页(page, 一般默认大小为4096bytes)对齐的，而且一定要判断mmap函数的返回值。

```c
#define MAP_SIZE 4096UL
#define MAP_MASK (MAP_SIZE - 1)

if((fd = open("/dev/mem", O_RDWR | O_SYNC)) == -1)
{
	FATAL;
}
printf("/dev/mem opened.\n"); 
fflush(stdout);
target = strtoul(argv[1], 0, 0);
map_base = mmap(0, MAP_SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, fd, target & ~MAP_MASK);
    if(map_base == (void *) -1)
	{
		FATAL;
	}
    printf("Memory mapped at address %p.\n", map_base); 
    fflush(stdout);
```


在driver中通过alloc_pages申请得到的page，将page的物理地址export到user space，但是user space拿到这个物理地址后并不能mmap成功。通过perror(“mmap”)，发现总是返回错误"Operation not permitted!"，后来发现是由于kernel对user space访问/dev/mem是有限制的，通过编译选项：CONFIG_STRICT_DEVMEM来限制user space 对物理内存的访问，这个选项的说明在arch/x86/Kconfig.debug中有说明：
```
config STRICT_DEVMEM
    bool "Filter access to /dev/mem"
    ---help---
      If this option is disabled, you allow userspace (root) access to all
      of memory, including kernel and userspace memory. Accidental
      access to this is obviously disastrous, but specific access can
      be used by people debugging the kernel. Note that with PAT support
      enabled, even in this case there are restrictions on /dev/mem
      use due to the cache aliasing requirements.
      If this option is switched on, the /dev/mem file only allows
      userspace access to PCI space and the BIOS code and data regions.
      This is sufficient for dosemu and X and all common users of
      /dev/mem.
      If in doubt, say Y.
```
只有在.config文件中设置CONFIG_STRICT_DEVMEM=n才能获得对整个memory的访问权限，在默认情况下，
CONFIG_STRICT_DEVMEM=y，这也就是之前mmap总是报错：“Operation not permitted”的原因。
设置这个选项后，编译kernel，然后运行tool，mmap还是返回错误：“Invalid argument”。后来查到还需要设置
编译选项CONFIG_X86_PAT=n，这个选项也是默认开启的，但是要关闭这个选项还需要开启CONFIG_EXPERT，
否则CONFIG_X86_PAT总是关不掉。
 
设置好这三个编译选项后，重新编译kernel，然后运行tool，发现kernel已经解除了对mmap的访问限制，可以
正确读取对应物理地址的内容了。
最后还可以通过修改内核源代码来实现，具体的源文件时在/drivers/char/目录下的mem.c文件
static inline int  range_is_allowed(unsigned long pfn, unsigned long size);


#######################################################################################

Attachment of Ref 3

I think I've found the issue -- it's to do with /dev/mem memory mapping protection on the x86.

Pl refer to this LWN article:"x86: introduce /dev/mem restrictions with a config option"http://lwn.net/Articles/267427/

CONFIG_NONPROMISC_DEVMEM

Now (i tested this on a recent 3.2.21 kernel), the config option seems to be called CONFIG_STRICT_DEVMEM.

I changed my kernel config:

```shell
$ grep DEVMEM .config
 CONFIG_STRICT_DEVMEM is not set
```
When the above prg was run with the previous kernel, with CONFIG_STRICT_DEVMEM SET:dmesg shows:

[29537.565599] Program a.out tried to access /dev/mem between 1000000->1001000.
[29537.565663] a.out[13575]: segfault at ffffffff ip 080485bd sp bfb8d640 error 4 in a.out[8048000+1000]
This is because of the kernel protection..

When the kernel was rebuilt (with the CONFIG_STRICT_DEVMEM UNSET) and the above prg was run :

```shell
$ ./a.out 
mmap failed: Invalid argument
```
This is because the 'offset' parameter is > 1 MB (invalid on x86) (it was 16MB).

After making the mmap offset to be within 1 MB:

```shell
$ ./a.out
```
addr: 0xb7758000
*addr: 138293760 
It works!See the above LWN article for details.

On x86 architectures with PAT support (Page Attribute Table), the kernel still prevents the mapping of DRAM regions. The reason for this as mentioned in the kernel source is:

This check is nedded to avoid cache aliasing when PAT is enabled
This check will cause a similar error to the one mentioned above. For example:

Program a.out tried to access /dev/mem between [mem 68200000-68201000].
This restriction can be removed by disabling PAT. PAT can be disabled by adding the "nopat" argument to the kernel command line at boot time.
