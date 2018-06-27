# 在 vagrant(Ubuntu 16.04) 中安裝 DPDK 與 OVS 紀錄筆記

## 前言
這是一個 Open vSwitch with DPDK 的安裝紀錄，OVS 可以在 userspace 完全使用 DPDK library 。

## 環境需求
- 如果是使用 Linux ，在 Linux 中 kernel 版本要大於 v3.0.0 版。
- DPDK v17.11.2
- 如果會使用到物理的 NIC 網卡，那就要使用有支援 DPDK 的網卡。可以參考 http://core.dpdk.org/supported/。(網卡查詢：`lspci | grep -i net`)

參考來源：http://docs.openvswitch.org/en/latest/intro/install/dpdk/  

## 前置作業
參考 http://doc.dpdk.org/guides/linux_gsg/sys_reqs.html 在系統需求上，有三個在編譯 DPDK 的前置作業。  
### 1. 在 x86 上 BIOS 設定的先決條件
不過大多數的平台，不需要特殊的 BIOS 設定，所以這邊跳過。
### 2. 需要的工具以及 Libraries
- GNU `make`
- coreutils: `cmp`, `sed`, `grep`, `arch`, etc.
- gcc v4.9 以上
- libc headers: gcc-multilib
- libnuma-devel: 用於處理 NUMA (Non Uniform Memory Access).
- Python, 版本 2.7+ 或 3.2+
### 3. 系統軟體
- Kernel 版本 >= 3.2 (檢查方法： `uname -r`)
- glibc >= 2.7 (檢查方法：`ldd --version`)
- Kernel config: 應該啟用 DPDK 的選項
    - HUGETLBFS
    - PROC_PAGE_MONITOR support
    - HPET and HPET_MMAP (如果有用到 HPET 才要做)

#### 4. 在 Linux 的環境中使用 Hugepages
因為 packet buffers 需要使用到 很大的 memory pool allocation ，所以需要使用到 Hugepage。這意味著 HUGETLBFS 選項需要在 running 的 kernel 中啟用。  
使用 hugepage allocations ，可以使用更少 page 並提高效能，可以減少 TLBs(Translation Lookaside Buffers)，減少虛擬的 page address 轉換到物理的 page address。如果沒有 hugepage，TLB 缺失率會變高，因為會發生在使用標準的 4k page size，降低。  

> 複習 or 補充：[Memory  管理的 page 架構](https://www.csie.ntu.edu.tw/~wcchen/asm98/asm/proj/b85506061/chap2/paging.html)、[hugetlbpage](https://github.com/torvalds/linux/blob/master/Documentation/admin-guide/mm/hugetlbpage.rst)、[Open vSwitch-DPDK 需要使用多少 Hugepage Memory](https://developers.redhat.com/blog/2018/03/16/ovs-dpdk-hugepage-memory/)  
> 其他：https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt, https://github.com/torvalds/linux/blob/master/Documentation/vm/hugetlbfs_reserv.rst

### 前置作業設定與套件安裝操作(在 Ubuntu 16.04 進行)
#### 1. 預先保留 Hugepages 給 DPDK 用
hugepage 的分配應該在開機後或是系統開機後越早做越好，因為要避免使用支離破碎的物理 Memory。(所以要使用完整的物理 Memory 空間)  
CPU 可以支援多大的 hugepage sizes 就直接看標誌決定。  
建議使用 1 GB hugepages。  
分為永久配置與暫時配置：  
- 永久配置(依據 http://docs.openvswitch.org/en/latest/intro/install/dpdk/#setup-hugepages 說明)
```sh
$ echo 'vm.nr_hugepages=1024' | sudo tee /etc/sysctl.d/hugepages.conf
```
> 參考：https://blog.csdn.net/shaoyunzhe/article/details/54614077

- 暫時配置 (依據 http://doc.dpdk.org/guides/linux_gsg/sys_reqs.html 說明)
```sh
$ echo 1024 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
```
如果有 NUMA 架構，需要額外再做另外的設定。  

設定完了可以檢查一下  
```sh
$ cat /proc/meminfo | grep Huge
AnonHugePages:         0 kB
HugePages_Total:    1024
HugePages_Free:     1024
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
```
#### 2. Using Hugepages with the DPDK
讓 DPDK 可以來使用這一塊 Memory
```sh
$ sudo mount -t hugetlbfs none /dev/hugepages
```
> 參考：https://docs.openstack.org/nova/pike/admin/huge-pages.html

### 3. 設定 kernel 變數在執行的時候
configure kernel parameters at runtime, enable writing `vm.nr_hugepages=1024` to variable
```sh
$ sudo sysctl -w vm.nr_hugepages=1024
```

#### 4. library 安裝
```sh
$ sudo apt update
$ sudo apt install -y vim git clang doxygen hugepages build-essential gcc-multilib libnuma-dev libpcap-dev inux-headers-`uname -r` dh-autoreconf libssl-dev libcap-ng-dev python python-pip
```

## 安裝 DPDK
### 1. 下載 DPDK 安裝包
```sh
$ wget --quiet https://fast.dpdk.org/rel/dpdk-17.11.2.tar.xz
$ sudo tar xf dpdk-17.11.2.tar.xz -C /usr/src/
```

PS. option: install pktgen-dpdk  
```sh
$ wget --quiet http://www.dpdk.org/browse/apps/pktgen-dpdk/snapshot/pktgen-3.4.9.tar.gz
$ sudo tar -zxf pktgen-3.4.9.tar.gz -C /usr/src/
```

### 2. 設定 DPDK 相關的環境變數
```sh
$ export DPDK_DIR=/usr/src/dpdk-stable-17.11.2
$ export DPDK_TARGET=x86_64-native-linuxapp-gcc
$ export DPDK_BUILD=$DPDK_DIR/$DPDK_TARGET
$ export LD_LIBRARY_PATH=$DPDK_DIR/x86_64-native-linuxapp-gcc/lib
```
- LD_LIBRARY_PATH: 如果 DPDK 是 shared library ，那這個環境變數是導出這個路徑給這個 lib 給 building OVS 用的


### 3. Build 與安裝 DPDK library
```sh
$ cd $DPDK_DIR && sudo make install T=$DPDK_TARGET DESTDIR=install
```

### 4. 設定 DPDK 為 shared library
```
$ sudo sed -i 's/CONFIG_RTE_BUILD_SHARED_LIB=n/CONFIG_RTE_BUILD_SHARED_LIB=y/g' ${DPDK_DIR}/config/common_base
```

## 設定 Linux Drivers
[UIO(Userspace IO)](https://github.com/torvalds/linux/tree/master/drivers/uio) 是一個 kernel module ，來設定 device ，他會 map device memory 到 user-space ，並且 register interrupts。  
### 1. load uio kernel module
```sh
$ sudo modprobe uio
```
### 2. insert kmod/igb_uio  module 到 Linux Kernel
因為DPDK 有支援 igb_uio ，而這個 module 可以在 kmod 這個子目錄下找到。  
(igb_uio 有支援 virtual function)  
```sh
$ sudo insmod ${DPDK_DIR}/x86_64-native-linuxapp-gcc/kmod/igb_uio.ko
```

PS. 與 UIO 相比，[VFIO](https://github.com/torvalds/linux/tree/master/drivers/vfio) driver 更加強大與安全(http://doc.dpdk.org/guides/linux_gsg/linux_drivers.html )。但這次不用。  

### 3. 開機後還是可以 load UIO 的設定
```
$ sudo ln -sf ${DPDK_DIR}/x86_64-native-linuxapp-gcc/kmod/igb_uio.ko /lib/modules/`uname -r`
$ sudo depmod -a
$ echo "uio" | sudo tee -a /etc/modules
$ echo "igb_uio" | sudo tee -a /etc/modules
```

### 4. 綁定 Network Ports 到 Kernel Modules
`usertools/dpdk-devbind.py` (a utility script) 可以用來綁定 port 到  igb_uio module ，這樣就可以使用 DPDK 囉！想知道更多可以使用 `--help` 或 `--usage`  
1. 查看 network ports 的狀態
```sh
$ ./usertools/dpdk-devbind.py --status                                                                   [17/1892]

Network devices using DPDK-compatible driver
============================================
<none>

Network devices using kernel driver
===================================
0000:00:03.0 '82540EM Gigabit Ethernet Controller 100e' if=enp0s3 drv=e1000 unused=igb_uio *Active*
0000:00:08.0 '82540EM Gigabit Ethernet Controller 100e' if=enp0s8 drv=e1000 unused=igb_uio *Active*

Other Network devices
=====================
<none>

Crypto devices using DPDK-compatible driver
===========================================
<none>

Crypto devices using kernel driver
==================================
<none>

Other Crypto devices
====================
<none>

Eventdev devices using DPDK-compatible driver
=============================================
<none>
...

$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 02:40:c1:fa:9b:f5 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::40:c1ff:fefa:9bf5/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:38:5e:e3 brd ff:ff:ff:ff:ff:ff
    inet 192.168.33.10/24 brd 192.168.33.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe38:5ee3/64 scope link
       valid_lft forever preferred_lft forever
```

2. 設定前先把 interface 改成 down
```
$ sudo ifconfig enp0s8 down
```
3. 綁定 device `enp0s8` 到 igb_uio
```sh
$ sudo ${DPDK_DIR}/usertools/dpdk-devbind.py --bind=igb_uio enp0s8
```
4. 再看一次 network interface
```sh
$ ./usertools/dpdk-devbind.py --status

Network devices using DPDK-compatible driver
============================================
0000:00:08.0 '82540EM Gigabit Ethernet Controller 100e' drv=igb_uio unused=e1000

Network devices using kernel driver
===================================
0000:00:03.0 '82540EM Gigabit Ethernet Controller 100e' if=enp0s3 drv=e1000 unused=igb_uio *Active*

$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 02:40:c1:fa:9b:f5 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::40:c1ff:fefa:9bf5/64 scope link
       valid_lft forever preferred_lft forever
```

## 安裝 OVS
### 1. 下載 OVS 安裝包
```sh
$ cd ~/
$ wget --quiet http://openvswitch.org/releases/openvswitch-2.9.0.tar.gz
$ sudo tar -zxf openvswitch-2.9.0.tar.gz -C /usr/src/
```

### 安裝 OVS
- 安裝必要的 pip 套件
```sh
$ sudo pip install six
```
- 設定 OVS_DIR 環境變數
```sh
$ export OVS_DIR=/usr/src/openvswitch-2.9.0
```
- 編譯 OVS
```sh
$ cd $OVS_DIR
$ ./boot.sh
$ CFLAGS='-march=native' ./configure --with-dpdk=$DPDK_BUILD
$ make && sudo make install
$ sudo mkdir -p /usr/local/etc/openvswitch
$ sudo mkdir -p /usr/local/var/run/openvswitch
$ sudo mkdir -p /usr/local/var/log/openvswitch
```
- 新增一個 ovsdb
```sh
$ sudo ovsdb-tool create /usr/local/etc/openvswitch/conf.db vswitchd/vswitch.ovsschema
```

### 關機後可以用 OVS
```
$ echo 'export PATH=$PATH:/usr/local/share/openvswitch/scripts' | sudo tee -a /root/.bashrc
```

### 移除
```
$ rm -rf /home/vagrant/openvswitch-2.9.0.tar.gz /home/vagrant/dpdk-17.11.2.tar.xz /home/vagrant/pktgen-3.4.9.tar.gz
```
  
==安裝成功==  
  
## Setup Open vSwitch
> http://docs.openvswitch.org/en/latest/intro/install/dpdk/#setup-ovs

### 1. 設定 Open vSwitch 的遠端 database server
```
$ ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock --remote=db:Open_vSwitch,Open_vSwitch,manager_options --pidfile --detach --log-file
```
### 2. 設定 ovs-vswitchd
> http://www.openvswitch.org/support/dist-docs/ovs-vswitchd.conf.db.5.txt
- init ovs-vswitchd
```sh
$ ovs-vsctl --no-wait init
```
- DPDK 的參數傳遞給 ovs-vswitchd ，透過 `Open_vSwitch` table 的 `other_config` 欄位。
    - the dpdk-init option must be set to either `true` or `try`
```sh
$ ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-init=true
```
- 從預先配置的 hugepage pool 中指定要給的 memory 量，on a per-socket basis。
```sh
$ ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-socket-mem="1024"
```
- 設定 cpu affinity 的 PMD  (Poll Mode Driver) threads 透過特定的 CPU mask
    - hex string
    - 最低位對應於第一個 CPU core。
```
$ ovs-vsctl --no-wait set Open_vSwitch . other_config:pmd-cpu-mask=0x2
```
- idle 的 flows 在 datapath 中 cached 的最大時間(in ms)
    - default is 10000
    - at least 500
```sh
$ ovs-vsctl --no-wait set Open_vSwitch . other_config:max-idle=30000
```
- [Open vSwitch daemon](http://www.openvswitch.org/support/dist-docs/ovs-vswitchd.8.html) 設定
```sh
$ ovs-vswitchd  unix:/usr/local/var/run/openvswitch/db.sock --pidfile --detach --log-file
```
- create a userspace bridge named br0 and add two dpdk ports to it
    - datapath_type: 是 datapath provider 的名稱。
        - userspace datapath type: `netdev`
        - kernel datapath type: `system`
    - dpdk-devargs: 指定物理 driver 的 PCI address 或是 virtual driver 的 virtual PMD. 可以用 `cd $DPDK_DIR &&./usertools/dpdk-devbind.py --status` 查。(在 VM 這邊查到的名稱是 0000:00:08.0)
```sh
$ ovs-vsctl add-br br0 -- set bridge br0 datapath_type=netdev
$ ovs-vsctl add-port br0 dpdk0 -- set Interface dpdk0 type=dpdk options:dpdk-devargs=0000:00:08.0
```
  
== 設定完成 ==  
使用 `htop` 可以看到第二顆 CPU core 用到 100%  


Ref: https://github.com/John-Lin/dpdk-ovs
