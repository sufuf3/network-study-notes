## DPDK

> page: 
> https://dpdk.org/
> https://dpdk.org/doc/guides/index.html

DPDK 全名是 data plane developmment kit
他是加速封包處理的 libraries 集合和 drivers。

他可以跑在任何的 processors 上，首先有支援的 CPU 是 intel x86 ，不過現在延伸到 IBM power 和 ARM 都有。

## Main libraries
- multicore framework
- huge page memory
- ring buffers
- poll-mode drivers for networking , crypto and eventdev

## libraries can be used to:
- receive and send packets within the minimum number of CPU cycles (usually less than 80 cycles)
- develop fast packet capture algorithms (tcpdump-like)
- run third-party fast path stacks

## Install
Ref: https://dpdk.org/doc/quick-start
```shell=
$ dmesg
[    1.043929] e1000: Intel(R) PRO/1000 Network Driver - version 7.3.21-k8-NAPI
[    1.043931] e1000: Copyright (c) 1999-2006 Intel Corporation.
$ sudo lshw -class network
  *-network
       description: Ethernet interface
       product: 82540EM Gigabit Ethernet Controller
       vendor: Intel Corporation
       physical id: 3
       bus info: pci@0000:00:03.0
       logical name: enp0s3
       version: 02
       serial: 08:00:27:5b:97:a5
       size: 1Gbit/s
       capacity: 1Gbit/s
       width: 32 bits
       clock: 66MHz
       capabilities: pm pcix bus_master cap_list ethernet physical tp 10bt 10bt-fd 100bt 100bt-fd 1000bt-fd autonegotiation
       configuration: autonegotiation=on broadcast=yes driver=e1000 driverversion=7.3.21-k8-NAPI duplex=full ip=10.0.2.15 latency=64 link=yes mingnt=255 multicast=yes port=twisted pair speed=1Gbit/s
       resources: irq:19 memory:f0000000-f001ffff ioport:d010(size=8)
```
get source code
```shell=
$ wget https://fast.dpdk.org/rel/dpdk-17.11.2.tar.xz
$ tar zxvf dpdk-17.11.2.tar.xz
```
Enable pcap (libpcap headers are required).
```sh
$ make config T=x86_64-native-linuxapp-gcc
$ sed -ri 's,(PMD_PCAP=).*,\1y,' build/.config
$ apt install libnuma-dev libpcap-dev=
```
build libraries
```shell=
$ make
```
Reserve huge pages memory.
```shell=
mkdir -p /mnt/huge
mount -t hugetlbfs nodev /mnt/huge
echo 64 > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
```
Run poll-mode driver test (with a cable between ports).
```shell=
build/app/testpmd -h
```

## Core Components
![](https://dpdk.org/doc/guides/_images/architecture-overview.svg)
- **EAL**(Environment Abstraction Layer)
    - responsible for gaining access to low-level resources(hardware, memory space...)
    - provides a generic interface that hides the environment specifics from the applications and libraries.
    - initialization routine to decide how to allocate these resources (that is, memory space, PCI devices, timers, consoles, and so on).
- **HPET**(High Precision Event Timer)
    - http://dpdk.org/doc/guides/linux_gsg/enable_func.html
- **rte_ring**(librte_ring)
    - Ring Manager 
    - provides a lockless multi-producer, multi-consumer FIFO API in a finite size table. 
    - A ring is used by the Memory Pool Manager (librte_mempool) and may be used as a general communication mechanism between cores and/or execution blocks connected together on a logical core.
    - https://dpdk.org/doc/guides/prog_guide/ring_lib.html#ring-library


Ref: 
https://github.com/DPDK/dpdk/tree/master/doc
http://www.c114.com.cn/topic/H3C1702/hot/list_DPDK.html
http://syswift.com/188.html
http://syswift.com/183.html
http://syswift.com/dpdk
http://syswift.com/224.html
https://feisky.gitbooks.io/sdn/dpdk/
