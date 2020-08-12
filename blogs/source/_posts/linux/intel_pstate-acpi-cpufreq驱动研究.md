---
title: intel_pstate/acpi-cpufreq驱动研究
tags: 
categories:
- linux
---
查看intel_pstate & cpu-freq驱动
Tag: v5.0
1.
linux/drivers/cpufreq/intel_pstate.c
	update_turbo_state(void)
	
	if (hwp_active)
		intel_pstate_hwp_enable(struct cpudata *cpudata)  
			0x773->0, 0x770->1

	Pn: core_get_min_pstate(void)  		CPU min MHz   // msr: 0x771 P0,P1,Pn HWP Performance Range Enumeration
		(0xce >> 40) & 0xFF = 0x0a = 10 -> 1000MHz

	P1: core_get_max_pstate_physical(void) 	CPU max MHz
		(0xce >> 8) & 0xFF  = 0x17 = 23 -> 2.3GHz

	P1: core_get_tdp_ratio(u64 plat_info) //1332
		tdp_msr = 0x64b(0x80000000) & 0x03 + 0x648 = 0x648
		rdmsr 0x648 = 0x17 = 23 -> 2.3GHz

	P1: core_get_max_pstate(void) //1365
		if(hwp_active) return tdp_ratio = 2.3GHz
		tar_levels = 0x64c(80000000) & 0xff = 0
		if (tdp_ratio - 1 == tar_levels) max_pstate = tar_levels

	P0: core_get_turbo_pstate(void)  //1400
		0x1ad & 255 = 0x27 = 39 -> 3.9GHz   msr寄存器0x1ad前7bit值

	intel_pstate_set_pstate(struct cpudata *cpu, int pstate) // 1453
		0x199
		0x198 Core Voltage (R/O) P-state core voltage can be computed by MSR_PERF_STATUS[37:32] * (float) 1/(2^13).

	intel_pstate_hwp_boost_up(struct cpudata *cpu) // 1516

	intel_pstate_hwp_boost_down(struct cpudata *cpu) // 1562

	intel_pstate_calc_avg_perf(struct cpudata *cpu) //1618
		core_avg_perf = (aperf << 14) / mperf

	get_avg_frequency(struct cpudata *cpu) // 1667
		(aperf << 14) / mperf * cpu_khz >> 14	  cpu_khz /* TSC clocks / usec, not used here */	

	get_avg_pstate(struct cpudata *cpu) //1672
		(aperf << 14) / mperf * max_pstate_physical >> 14	

	static struct cpufreq_driver *default_driver = &intel_pstate; // 2327
		默认是intel_pstate驱动
		
	early_param("intel_pstate", intel_pstate_setup); //2682
		读取 /etc/default/grub 引导文件中的参数

2.
linux/drivers/cpufreq/acpi-cpufreq.c

3.
linux/drivers/cpufreq/cppc_cpufreq.c

4.
linux/drivers/idle/intel_idle.c
