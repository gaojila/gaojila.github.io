# Vmware-linux虚拟机的磁盘扩容


# 背景

40GB(sys)256GB\*(data)，合并成为一个 lvm 逻辑分区 ubuntu-root

# 目标

最小代价实现 lvm 扩容 64GB

# 方法

#### 新建一块 64GB 虚拟硬盘，合并到 ubuntu-root 逻辑分区

- 优点：风险最小
- 缺点：多一块 64GB 虚拟磁盘

#### 新建一块 320GB 虚拟磁盘，转移原 256GB 数据后，删除 256GB 虚拟磁盘

- 优点：只保留一块 320GB 虚拟磁盘
- 缺点：代价高

#### 增加 256GB 虚拟磁盘容量，直接动态扩容到 320GB

- 优点：技能 GET
- 缺点：未知风险高

# 操作

#### 在 vClient 修改虚拟磁盘（假定未 sdb）的容量，从 256GB 调整到 320GB

#### 重启服务器（不建议）、或强制重新扫描磁盘（具体那个实际有效暂不清楚）

- echo 1 > /sys/class/scsi_disk/2\:0\:0\:0/device/rescan
- echo '- - -' > /sys/class/scsi_host/host2/scan
- echo 1 > /sys/block/sdb/device/rescan (确认有效)
- partprobe -s

#### 分两种情况，A）sdb 未分区、直接整盘加入 lvm 的情况

- 确认 sdb 磁盘容量已增加 (fdisk)
- 执行 pvresize /dev/sdb 对 LVM 的物理分区扩容
- 对 LVM 的逻辑分区扩容(略)

#### 第二种情况，B）sdb 已分区、某个（些）分区加入 lvm 的情况（假如 sdb5）

- 确认 sdb 磁盘容量已增加 (fdisk)
- 通过 parted /dev/sdb 对 sdb5 分区扩容 (注意，如果分区是在扩展分区中，需要先把扩展分区扩容)

```bash
root@nj-otter-en-qa:/opt/logstash# parted /dev/sda
GNU Parted 2.3
Using /dev/sda
Welcome to GNU Parted! Type help'' to view a list of commands.
parted)( p
Model: VMware Virtual disk (scsi)
Disk /dev/sda: 68.7GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type      File system  Flags
1      1049kB  256MB   255MB   primary   ext2         boot
2      257MB   42.9GB  42.7GB  extended
5      257MB   42.9GB  42.7GB  logical                lvm

(parted) resizepart 2
End?  [68.0GB]? 68.7GB
(parted) p
Model: VMware Virtual disk (scsi)
Disk /dev/sda: 68.7GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type      File system  Flags
 1      1049kB  256MB   255MB   primary   ext2         boot
 2      257MB   68.7GB  68.4GB  extended
 5      257MB   42.9GB  42.7GB  logical                lvm

(parted) resizepart 5
End?  [42.9GB]? 68.7GB
Error: Error informing the kernel about modifications to partition /dev/sda5 --
Invalid argument.  This means Linux won't know about any changes you made to
/dev/sda5 until you reboot -- so you shouldn't mount it or use it in any way
before rebooting.
Ignore/Cancel? cancel
Error: Partition(s) 5 on /dev/sda have been written, but we have been unable to
inform the kernel of the change, probably because it/they are in use.  As a
result, the old partition(s) will remain in use.  You should reboot now before
making further changes.
Ignore/Cancel? Cancel
(parted) p
Model: VMware Virtual disk (scsi)
Disk /dev/sda: 68.7GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type      File system  Flags
 1      1049kB  256MB   255MB   primary   ext2         boot
 2      257MB   68.7GB  68.4GB  extended
 5      257MB   68.7GB  68.4GB  logical                lvm
```

#### 执行 pvresize /dev/sdb5 对 LVM 的物理分区扩容

#### 对 LVM 的逻辑分区扩容(略)

