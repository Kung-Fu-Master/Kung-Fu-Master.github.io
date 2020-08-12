---
title: 用户态和内核共享内存----使用 /dev/mem & mmap
tags: 
categories:
- linux
---
想法的来源是看到chinaunix上有人转载了wheelz的博客，但是wheelz的代码在我的实验平台上是不能正常工作的，可能是wheelz的代码太过久远，我试验的内核版本是：3.4.13。wheelz的源代码如下：
// 内核模块
#include <linux/config.h>
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/mm.h>
MODULE_LICENSE("GPL");
MODULE_AUTHOR("Wheelz");
MODULE_DESCRIPTION("mmap demo");
static unsigned long p = 0;
static int __init init(void)
{
        //分配共享内存（一个页面）
        p = __get_free_pages(GFP_KERNEL, 0);
        SetPageReserved(virt_to_page(p));
        printk("<1> p = 0x%08x\n", p); 
    //p是内核中的虚拟地址  
        //在共享内存中写上一个字符串
        strcpy(p, "Hello world!\n");
        return 0;
}
static void __exit fini(void)
{
        ClearPageReserved(virt_to_page(p));
        free_pages(p, 0);        
}
module_init(init);
module_exit(fini);
----------------------------------------------------------------------------------------------------------------------------------------
// 用户态程序
#include <sys/mman.h> 
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h> 
#include <stdio.h> 
#define PAGE_SIZE (4*1024)
#define PAGE_OFFSET                0xc0000000
#define KERNEL_VIRT_ADDR        0xc5e3c000
int main() 
{ 
        char *buf; 
        int fd; 
        unsigned long phy_addr; 
fd=open("/dev/mem",O_RDWR); 
if(fd == -1)
                perror("open");
        phy_addr=KERNEL_VIRT_ADDR - PAGE_OFFSET; 
        //此处不太懂，不能理解物理地址phy_addr的计算方法
        buf=mmap(0, PAGE_SIZE, 
                PROT_READ|PROT_WRITE, MAP_SHARED, 
                fd, phy_addr); 
        if(buf == MAP_FAILED)
                perror("mmap");
puts(buf);//打印共享内存的内容
        munmap(buf,PAGE_SIZE); 
        close(fd); 
        return 0; 
} 
在网上找了一些资料，导致这段代码不工作的原因可能有一下几个：
（1）在编译内核时设置了CONFIG_STRICT_DEVMEM（某些版本中是CONFIG_NONPROMISC_DEVMEM），应该将此设置删除。
（2）请求的地址没有通过内核中devmem_is_allowed函数对/dev/mem的保护。
（3）物理地址phy_addr计算错误。（PS：wheelz的计算方法是怎么得到的？）
 
我对上面的几个问题一一做了修改：
（1）修改了.config文件
# CONFIG_STRICT_DEVMEM is not set
（２）重写arch/x86/mm/init.c下的devmem_is_allowed函数，这里我没有做太细致的修改，只是让函数一直返回1。当然这可能会存在一些问题。

/*
 * devmem_is_allowed() checks to see if /dev/mem access to a certain address
 * is valid. The argument is a physical page number.
 *
 *
 * On x86, access has to be given to the first megabyte of ram because that area
 * contains bios code and data regions used by X and dosemu and similar apps.
 * Access has to be given to non-kernel-ram areas as well, these contain the PCI
 * mmio resources as well as potential bios/acpi data regions.
 */
int devmem_is_allowed(unsigned long pagenr)
{
return 1;
        if (pagenr <= 256)
                return 1;
        if (iomem_is_exclusive(pagenr << PAGE_SHIFT))
                return 0;
        if (!page_is_ram(pagenr))
                return 1;
        return 0;
}
（3）修改物理地址的计算，这里我们直接使用内核中提供的转换函数virt_to_phy()或者__pa()。

// 内核模块
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/mm.h>
MODULE_LICENSE("GPL");
MODULE_AUTHOR("godjesse");
MODULE_DESCRIPTION("mmap demo");
static unsigned long p = 0;
static unsigned long pp = 0;
static int __init init(void)
{
        p = __get_free_pages(GFP_KERNEL, 0);
if(!p)
        {
                printk("Allocate memory failure!/n");
        }
        else
        {
                SetPageReserved(virt_to_page(p));
// 使用virt_to_phys计算物理地址，供用户态程序使用
                pp = (unsigned long)virt_to_phys((void *)p);
                printk("<1> page : pp = 0x%lx\n",pp);
        }
        strcpy((char *)p, "Hello world !\n");
        return 0;
}
static void __exit fini(void)
{
printk("The content written by user is: %s/n", (unsigned char *)p);
        ClearPageReserved(virt_to_page(p));
        free_pages(p, 0);
        printk(" exit \n");
}
module_init(init);
module_exit(fini);
---------------------------------------------------------------------------------------------------------------------------------------
// 用户态程序
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>
#include <string.h>
//hard coding read after the module installed
#define KERNEL_PHY_ADDR  0x3737c000
int main()
{
char *buf;
        int fd;
        unsigned long phy_addr;
        int  pagesize = getpagesize();
phy_addr=KERNEL_PHY_ADDR;
fd=open("/dev/mem",O_RDWR);
if(fd == -1)
                perror("open");
buf=mmap(0, pagesize, PROT_READ|PROT_WRITE, MAP_SHARED, fd, phy_addr);
if(buf == MAP_FAILED)
        {
                perror("mmap");
        }
printf("buf : %s\n",buf);
// test the write 
        buf[0] = 'X';
munmap(buf,pagesize);
        close(fd);
        return 0;
}

 经过这些修改后，demo可以正常工作。
上文中提到修改devmem_is_allowed实际上是存在问题的，存在其他一些较为优雅的方法，如某牛人写的博客：bypassing devmem_is_allowed with kernel probes，博客链接：
http://www.libcrack.so/2012/09/02/bypassing-devmem_is_allowed-with-kprobes/
相关资料：
http://stackoverflow.com/questions/11891979/accessing-mmaped-dev-mem 

From <https://www.cnblogs.com/godjesse/archive/2012/11/23/2784093.html> 
