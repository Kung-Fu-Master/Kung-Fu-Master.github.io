---
title: perf
tags: 
categories:
- linux
---
/root/spdk/examples/nvme/perf/perf -q 64 -s 1024 -w write -t 10 -c 0x01 -o 4096 -D -LL -r 'trtype:PCIe traddr:0000:5e:00.0' 
