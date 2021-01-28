# linux系统常用的参数优化


## Ulimit 配置

linux 操作系统默认只能打开 1024 个文件，打开的文件超过这个数发现程序会有“too many open files”的错误，1024 对于大数据系统来说显然是不够的，如果不设置，基本上整个大数据系统是“不可用的”，根本不能用于生产环境。
配置方法如下：

```bash
echo "* soft nofile 128000" >> /etc/security/limits.conf
echo "* hard nofile 128000" >> /etc/security/limits.conf
echo "* soft nproc 128000" >> /etc/security/limits.conf
echo "* hard nproc 128000" >> /etc/security/limits.conf
```

【修改建议：强烈建议修改，无影响】

## swap 分区配置

让系统尽量不使用 swap，如果按照默认配置为 60，则容易导致内存还够的情况下使用 swap，有可能导致 jvm 的 gc 回收处于 swap 的内存，造成一系列超时问题。
不是不能再使用 swap，只是尽量不使用 swap。

```bash
echo "vm.swappiness=1" >> /etc/sysctl.conf
sysctl -p
sysctl -a|grep swappiness
```

【修改建议：强烈建议修改，无影响】

## 内存映射数量限制问题

如果 solr 内存映射过多，会超出系统限制的个数 65530，导致 solr 问题。

```bash
vi /etc/sysctl.conf
vm.max_map_count=262144
sysctl -p
```

【修改建议：强烈建议修改，无影响】

## 监听队列大小

tcp 连接的时候 listen 监听队列的大小默认为 128.

```bash
echo " net.core.somaxconn = 32768 " >> /etc/sysctl.conf
sysctl -p
sysctl -a|grep somaxconn
```

【修改建议： 一般，无影响】

## 透明大页问题

在 centos7 的系统版本中，透明大页这种本来为提高性能的手段，会系统负载高的时候，造成系统反复重启，所以建议关闭。

```bash
1）查看是否启用：
[root@localhost ~]# cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never

[always]为启动。
2）停止方法：
1、第一种方法：对于centos7来说：【临时修改可以直接用下面两条命令】
更改：/etc/rc.d/rc.local
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
修改权限： chmod +x /etc/rc.d/rc.local

2、第二种方法：
修改 /etc/grub.conf 重启后生效。
添加：transparent_hugepage=never
举个例子：
For example:

title Oracle Linux Server (2.6.32-300.25.1.el6uek.x86_64)
root (hd0,0)
kernel /vmlinuz-2.6.32-300.25.1.el6uek.x86_64 ro root=LABEL=/ transparent_hugepage=never
initrd /initramfs-2.6.32-300.25.1.el6uek.x86_64.img
```

【修改建议：建议修改，对系统性能有点影响】

## 内存分配策略 overcommit_memory

```
[root@localhost ~]# cat /proc/sys/vm/overcommit_memory
0
内核参数overcommit_memory 它是 内存分配策略 可选值：0、1、2。
0， 表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，并把错误返回给应用进程。
1， 表示内核允许分配所有的物理内存，而不管当前的内存状态如何。
2， 表示内核允许分配超过所有物理内存和交换空间总和的内存
【建议设置为0，目前环境为0 不用修改】
```

## NUMA 参数问题

numa 为一种架构模式，就是内存和 cpu 组绑定，提升总线通信带宽。配置不当容易造成明明内存很多，但是却在使用 swap 问题。

```
1、查看是否开启NUMA
通过命令： grep -i numa /var/log/dmesg
如果输出:No NUMA configuration found 则没有开启，否则是开启了NUMA
我们主机显示：
[ 3.175740] pci_bus 0000:00: on NUMA node 0
[ 3.180438] pci_bus 0000:10: on NUMA node 1
[ 3.185192] pci_bus 0000:20: on NUMA node 2
[ 3.189191] pci_bus 0000:30: on NUMA node 3
[ 3.191694] pci_bus 0000:40: on NUMA node 4
[ 3.194062] pci_bus 0000:50: on NUMA node 5
[ 3.198240] pci_bus 0000:60: on NUMA node 6
[ 3.200434] pci_bus 0000:70: on NUMA node 7

表明开启了NUMA。

2、查看分配策略
cat /proc/sys/vm/zone_reclaim_mode

目前环境为0 ，建议为0 不用修改。
当某个节点可用内存不足时：
1、如果为0的话，那么系统会倾向于从其他节点分配内存
2、如果为1的话，那么系统会倾向于从本地节点回收Cache内存多数时候
```

## 时钟同步

使用 chronyc 进行时钟同步

## 关闭 SELINUX

```bash
1）查看：
/etc/selinux/config
2）修改
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

## 禁用 IPV6

```bash
echo -e "NETWORKING_IPV6=no" >> /etc/sysconfig/network
echo -e "alias net-pf-10 off\noptions ipv6 disable=1" > /etc/modprobe.d/disable_ipv6.conf
systemctl restart network
```

