# greenplum安装


# 所有节点操作

## 所有节点安装 greenplum

```bash
add-apt-repository ppa:greenplum/db
apt-get update
apt-get install greenplum-db-oss
```

## 将所有节点添加到 hosts 文件中

```bash
172.16.132.134  gpmaster
172.16.132.136  gpslave
172.16.132.135  gpsegment1
172.16.132.137  gpsegment2
```

## 添加 gpadmin 用户

```bash
groupadd gpadmin
useradd -m -g gpadmin -s /bin/bash gpadmin
```

## 更改系统参数

- vim /etc/sysctl.conf

```bash
kernel.shmmax = 500000000
kernel.shmmni = 4096
kernel.shmall = 4000000000
kernel.sem = 250 512000 100 2048
kernel.sysrq = 1
kernel.core_uses_pid = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.msgmni = 2048
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.conf.all.arp_filter = 1
net.ipv4.ip_local_port_range = 1025 65535
net.core.netdev_max_backlog = 10000
net.core.rmem_max = 2097152
net.core.wmem_max = 2097152
vm.overcommit_memory = 2
vm.overcommit_ratio = 95
net.ipv4.ip_forward = 0
```

- vim /etc/security/limits.conf

```bash
* soft nofile 65536
* hard nofile 65536
* soft nproc 131072
* hard nproc 131072
```

## 设置 ntp

```bash
echo "server pgmaster" >> /etc/ntp.conf
systemctl restart ntpd
```

## 设置 segments 数据目录存储参数(仅在 segment 机器上执行即可)

```bash
blockdev --setra 16384 /dev/sda
```

## 重启机器

```bash
reboot
```

# master 节点操作

## 创建 data 目录

```bash
mkdir -p /data/master
chown -R gpadmin:gpadmin /data
```

## 配置环境变量

```bash
echo "source /opt/gpdb/greenplum_path.sh" >> /home/gpadmin/.bashrc
source .bashrc
```

## 初始化 greenplum(gpadmin 用户家目录下)

- 新建 allhosts

```bash
gpmaster
gpslave
gpsegment1
gpsegment2
```

- 新建 seghosts

```bash
gpsegment1
gpsegment2
```

- 修改初始化配置

```bash
cp /opt/gpdb/docs/cli_help/gpconfigs/gpinitsystem_config . && vim gpinitsystem_config
(
declare -a DATA_DIRECTORY=(/data/primary )
MASTER_HOSTNAME=gpmaster
MASTER_DIRECTORY=/data/master
MIRROR_PORT_BASE=50000
REPLICATION_PORT_BASE=41000
MIRROR_REPLICATION_PORT_BASE=51000
)
```

- 配置免密钥登录

```bash
gpssh-exkeys -f allhosts
```

- 在各节点新建数据目录

```bash
gpssh -h gsdw-standby -e 'mkdir -p /data/master'
gpssh -f hostfile_seg -e 'mkdir -p /data/primary'
gpssh -f hostfile_seg -e 'mkdir -p /data/mirror'
```

- 补充参数设置(root 用户下执行)

```bash
echo "RemoveIPC=no" >> /etc/systemd/logind.conf
service systemd-logind restart
```

- 执行数据库初始化

```bash
gpinitsystem -c gpinitsystem_config -h seghosts
gpinitstandby -s gpslave
```

- 将数据目录加入到环境变量

```bash
echo "export MASTER_DATA_DIRECTORY=/data/master/gpseg-1/" >> /home/gpadmin/.bashrc
```

# 备注

- pgsql postgres(greenplum 没有 pgadmin 库不能 psql 直接进入数据库)

