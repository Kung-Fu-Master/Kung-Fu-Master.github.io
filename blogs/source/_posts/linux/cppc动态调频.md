---
title: cppc动态调频
tags: 
categories:
- linux
---
CPPC的全称是Collaborative Processor Performance Control
CPC的全称是Per cpu table called，是bios提供的一组acpi表(ACPI表示高级配置和电源管理接口（Advanced Configuration and Power Management Interface))，用于设置cpu的频率。这组acpi表如下：

```
1. /*
2. * An example CPC table looks like the following.
3. *
4. *	Name(_CPC, Package()
5. *			{
6. *			17,
7. *			NumEntries
8. *			1,
9. *			// Revision
10. *			ResourceTemplate(){Register(PCC, 32, 0, 0x120, 2)},
11. *			// Highest Performance
12. *			ResourceTemplate(){Register(PCC, 32, 0, 0x124, 2)},
13. *			// Nominal Performance
14. *			ResourceTemplate(){Register(PCC, 32, 0, 0x128, 2)},
15. *			// Lowest Nonlinear Performance
16. *			ResourceTemplate(){Register(PCC, 32, 0, 0x12C, 2)},
17. *			// Lowest Performance
18. *			ResourceTemplate(){Register(PCC, 32, 0, 0x130, 2)},
19. *			// Guaranteed Performance Register
20. *			ResourceTemplate(){Register(PCC, 32, 0, 0x110, 2)},
21. *			// Desired Performance Register
22. *			ResourceTemplate(){Register(SystemMemory, 0, 0, 0, 0)},
23. *			..
24. *			..
25. *			..
26. *
27. *		}
28. * Each Register() encodes how to access that specific register.
29. * e.g. a sample PCC entry has the following encoding:
30. *
31. *	Register (
32. *		PCC,
33. *		AddressSpaceKeyword
34. *		8,
35. *		//RegisterBitWidth
36. *		8,
37. *		//RegisterBitOffset
38. *		0x30,
39. *		//RegisterAddress
40. *		9
41. *		//AccessSize (subspace ID)
42. *		0
43. *		)
44. *	}
45. */
```

那cppc表具体要怎么工作呢？具体在driver/cpufreq/cppc_cpufreq.c中。
这里的cppc_cpufreq_init是入口函数，这个函数向cpufreq的framework注册了一个可以调频的cpu driver

```shell
static int __init cppc_cpufreq_init(void)
{
ret = cpufreq_register_driver(&cppc_cpufreq_driver);
if (ret)
goto out;
}
static struct cpufreq_driver cppc_cpufreq_driver = {
.flags = CPUFREQ_CONST_LOOPS,
.verify = cppc_verify_policy,
 .target = cppc_cpufreq_set_target,
 .get = cppc_cpufreq_get_rate,
 .init = cppc_cpufreq_cpu_init,
 .stop_cpu = cppc_cpufreq_stop_cpu,
 .name = "cppc_cpufreq",
 };
 //cppc_cpufreq_driver 最终的函数就是target，最终cpu调频就是通过target 这个回调函数来实现
 static int cppc_cpufreq_set_target(struct cpufreq_policy *policy,
 unsigned int target_freq,
 unsigned int relation)
 {
 struct cppc_cpudata *cpu;
 struct cpufreq_freqs freqs;
 u32 desired_perf;
 int ret = 0;
 cpu = all_cpu_data[policy->cpu];
 //得到要设置的频率
 desired_perf = cppc_cpufreq_khz_to_perf(cpu, target_freq);
 /* Return if it is exactly the same perf */
 if (desired_perf == cpu->perf_ctrls.desired_perf)
 return ret;
 cpu->perf_ctrls.desired_perf = desired_perf;
 freqs.old = policy->cur;
 freqs.new = target_freq;
 cpufreq_freq_transition_begin(policy, &freqs);
 //通过acpi 提供的的接口来设置cpu 频率
 ret = cppc_set_perf(cpu->cpu, &cpu->perf_ctrls);
 cpufreq_freq_transition_end(policy, &freqs, ret != 0);
 if (ret)
 pr_debug("Failed to set target on CPU:%d. ret:%d\n",
 cpu->cpu, ret);
 return ret;
 }
```

这里的cppc_set_perf实现在driver/acpi/cppc_acpi.c中实现，通过这个接口可以通过固件来设置cpu频率。
