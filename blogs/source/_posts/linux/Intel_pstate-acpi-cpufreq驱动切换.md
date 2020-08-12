---
title: Intel_pstate/acpi-cpufreq驱动切换 
type: 
categories:
- linux
---

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
