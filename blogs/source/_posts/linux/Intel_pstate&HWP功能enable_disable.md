---
title: Intel_pstate & HWP功能enable/disable
tags: 
categories:
- linux
---

linux/drivers/cpufreq/intel_pstate.c

From <https://github.com/torvalds/linux/blob/v5.0/drivers/cpufreq/intel_pstate.c> 


Intel_pstate & acpi-cpufreq 驱动切换
By default it is intel_pstate driver
               Change intel_pstate driver to acpi-cpufreq as follow:

    /etc/default/grub, 
change
GRUB_CMDLINE_LINUX="***quiet"
to
GRUB_CMDLINE_LINUX="***quiet intel_pstate=disable"
then
UEFI 系统上的指令是 grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
after reboot, then acpi-cpufreq driver willbe used.


查看HWP功能是否开启：
#rdmsr 0x770, 读取1则enable， 读取0则disable
关闭HWP功能：
Vim /etc/default/grub
GRUB_CMDLINE_LINUX = "*** intel_pstate=no_hwp"

intel_pstate=disable	disable intel_pstate 驱动
intel_pstate=passive	Passive mode enabled, default_driver = &intel_cpufreq
	机器重启后cpupower frequency-info查看驱动确实是intel_cpufreq 
	  current CPU frequency: Unable to call hardware
	  current CPU frequency: 1000 MHz (asserted by call to kernel)
intel_pstate=no_hwp	HWP disabled, 重启后执行cpupower frequency-info命令查看
	  current CPU frequency: Unable to call hardware
	  current CPU frequency: 1000 MHz (asserted by call to kernel)
intel_pstate=force	
intel_pstate=hwp_only	
intel_pstate=per_cpu_perf_limits	
intel_pstate=support_acpi_ppc	acpi_ppc = true
