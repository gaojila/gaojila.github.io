# mysql高可用架构mha安装


# IP 规划

```bash
10.1.1.22(master)
10.1.1.21(master ca)
10.1.1.20(slave)
172.18.36.75(manager)
```

---

# 安装前准备工作

## master 与 slave 之间配置 ssh 免密钥登录

## mha 依赖与主从复制，请提前配置好主从复制

---

# 安装 mha node

## 安装 perl 环境

### 下载 cpanm

```bash
wget http://xrl.us/cpanm --no-check-certificate
```

### playbook template 模板

```bash
#!/bin/bash

#================================================================
#   Copyright (C) 2019 Sangfor Ltd. All rights reserved.
#
#   文件名称：install.sh
#   创 建 者：luwenzheng
#   邮    箱：redgaojila@gmail.com
#   创建日期：2019年04月23日
#   描    述：
#
#================================================================
#!/bin/bash
mv /var/tmp/{{ lookup('pipe', 'date +%m%d') }}/cpanm /usr/bin
chmod 755 /usr/bin/cpanm
cat > /root/list << EOF
install DBD::mysql
EOF
for package in `cat /root/list`
do
    cpanm $package
done

```

## node 安装

### 下载 node deb 包

```bash
wget https://github.com/yoshinorim/mha4mysql-node/releases/download/v0.58/mha4mysql-node_0.58-0_all.deb
```

### playbook 安装

```yaml
---
- name: 建立备份目录
  file: >
    path="/var/tmp/{{ lookup('pipe', 'date +%m%d') }}"
    state=directory

- name: copy 安装包
  copy: >
    src=/home/luwenzheng/ansible/roles/gens.mysqlmha/files/mha4mysql-node_0.58-0_all.deb
    dest=/var/tmp/{{ lookup('pipe', 'date +%m%d') }}

- name: 安装node节点
  shell: dpkg -i /var/tmp/{{ lookup('pipe', 'date +%m%d') }}/mha4mysql-node_0.58-0_all.deb
```

### 安装完成后可在/usr/bin 下有以下文件

```bash
root@nj-lab-0-it:~# dpkg -L mha4mysql-node
/.
/usr
/usr/bin
/usr/bin/purge_relay_logs
/usr/bin/filter_mysqlbinlog
/usr/bin/save_binary_logs
/usr/bin/apply_diff_relay_logs
/usr/share
/usr/share/man
/usr/share/man/man1
/usr/share/man/man1/save_binary_logs.1p.gz
/usr/share/man/man1/filter_mysqlbinlog.1p.gz
/usr/share/man/man1/apply_diff_relay_logs.1p.gz
/usr/share/man/man1/purge_relay_logs.1p.gz
/usr/share/perl5
/usr/share/perl5/MHA
/usr/share/perl5/MHA/BinlogPosFinderElp.pm
/usr/share/perl5/MHA/BinlogPosFindManager.pm
/usr/share/perl5/MHA/BinlogPosFinderXid.pm
/usr/share/perl5/MHA/BinlogPosFinder.pm
/usr/share/perl5/MHA/BinlogHeaderParser.pm
/usr/share/perl5/MHA/NodeConst.pm
/usr/share/perl5/MHA/NodeUtil.pm
/usr/share/perl5/MHA/SlaveUtil.pm
/usr/share/perl5/MHA/BinlogManager.pm
/usr/share/doc
/usr/share/doc/mha4mysql-node
/usr/share/doc/mha4mysql-node/changelog.Debian.gz
/usr/share/doc/mha4mysql-node/AUTHORS
/usr/share/doc/mha4mysql-node/README
/usr/share/doc/mha4mysql-node/copyright
```

---

## mha manager 安装

### 下载 manager deb 包

```bash
wget https://github.com/yoshinorim/mha4mysql-node/releases/download/v0.58/mha4mysql-manager_0.58-0_all.deb
```

### playbook 安装

```yaml
---
- name: 创建备份目录和配置文件目录
  file: >
    path={{ item }}
    state=directory
  with_items:
    - /var/tmp/{{ lookup('pipe', 'date +%m%d') }}
    - /usr/local/masterha/app1
    - /etc/masterha

- name: 创建配置文件
  file: >
    path={{ item }}
    state=touch
  with_items:
    - /etc/masterha/app1/master_ip_failover
    - /etc/masterha/app1/master_ip_online_change
    - /etc/masterha/app1/send_report
    - /etc/masterha/app1/power_manager
    - /etc/masterha/app1/app1.conf

- name: copy 安装包
  copy: >
    src=/home/luwenzheng/ansible/roles/gens.mysqlmha/files/mha4mysql-manager_0.58-0_all.deb
    dest=/var/tmp/{{ lookup('pipe', 'date +%m%d') }}

- name: 安装master
  shell: dpkg -i /var/tmp/{{ lookup('pipe', 'date +%m%d') }}/mha4mysql-manager_0.58-0_all.deb
```

### 安装完成后会有以下文件

```bash
root@nj-lab-2-it:/var/tmp/0423# dpkg -L mha4mysql-manager
/.
/usr
/usr/bin
/usr/bin/masterha_check_status
/usr/bin/masterha_check_ssh
/usr/bin/masterha_master_monitor
/usr/bin/masterha_manager
/usr/bin/masterha_master_switch
/usr/bin/masterha_stop
/usr/bin/masterha_secondary_check
/usr/bin/masterha_check_repl
/usr/bin/masterha_conf_host
/usr/share
/usr/share/man
/usr/share/man/man1
/usr/share/man/man1/masterha_master_monitor.1p.gz
/usr/share/man/man1/masterha_manager.1p.gz
/usr/share/man/man1/masterha_secondary_check.1p.gz
/usr/share/man/man1/masterha_check_ssh.1p.gz
/usr/share/man/man1/masterha_master_switch.1p.gz
/usr/share/man/man1/masterha_check_repl.1p.gz
/usr/share/man/man1/masterha_stop.1p.gz
/usr/share/man/man1/masterha_check_status.1p.gz
/usr/share/man/man1/masterha_conf_host.1p.gz
/usr/share/perl5
/usr/share/perl5/MHA
/usr/share/perl5/MHA/ManagerAdmin.pm
/usr/share/perl5/MHA/Server.pm
/usr/share/perl5/MHA/MasterRotate.pm
/usr/share/perl5/MHA/Config.pm
/usr/share/perl5/MHA/ManagerAdminWrapper.pm
/usr/share/perl5/MHA/ServerManager.pm
/usr/share/perl5/MHA/HealthCheck.pm
/usr/share/perl5/MHA/ManagerConst.pm
/usr/share/perl5/MHA/DBHelper.pm
/usr/share/perl5/MHA/SSHCheck.pm
/usr/share/perl5/MHA/FileStatus.pm
/usr/share/perl5/MHA/ManagerUtil.pm
/usr/share/perl5/MHA/MasterFailover.pm
/usr/share/perl5/MHA/MasterMonitor.pm
/usr/share/doc
/usr/share/doc/mha4mysql-manager
/usr/share/doc/mha4mysql-manager/changelog.Debian.gz
/usr/share/doc/mha4mysql-manager/AUTHORS
/usr/share/doc/mha4mysql-manager/README
/usr/share/doc/mha4mysql-manager/copyright
```

### 文件描述

```bash
masterha_check_repl             检查MySQL复制状况
masterha_check_ssh              检查MHA的SSH配置状况
masterha_check_status           检测当前MHA运行状态
masterha_conf_host              添加或删除配置的server信息
masterha_manger                 启动MHA
masterha_master_monitor         检测master是否宕机
masterha_master_switch          控制故障转移（自动或者手动）
```

---

## mha manager 配置

### 创建 mha 工作目录

```bash
mkdir /etc/masterha
```

### app1.cnf 配置

```bash
[server default]
manager_log=/var/log/masterha/app1-manager.log
manager_workdir=/etc/masterha/app1
master_binlog_dir=/var/lib/mysql
password=********
ping_interval=5
remote_workdir=/tmp
repl_password=********
repl_user=root
secondary_check_script=/usr/bin/masterha_secondary_check -s 10.1.1.20 -s 10.1.1.21 --user=root --master_host=10.1.1.22 --master_ip=10.1.1.22 --master_port=3306
shutdown_script=""
ssh_user=root
user=mha_rep

[server2]
candidate_master=1
check_repl_delay=0
hostname=10.1.1.21
port=3306

[server3]
hostname=10.1.1.20
port=3306

```

