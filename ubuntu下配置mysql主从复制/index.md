# ubuntu下配置mysql主从复制


# 主服务器配置

1. 修改 Mysql 配置

```bash
vi /etc/msyql/mysql.conf.d/msyqld.conf
```

2. 在[msyqld]中加

```bash
#主数据库端ID号
server_id = 1
 #开启二进制日志
log-bin = mysql-bin
#需要复制的数据库名，如果复制多个数据库，重复设置这个选项即可
binlog-do-db = db
#将从服务器从主服务器收到的更新记入到从服务器自己的二进制日志文件中
log-slave-updates
#控制binlog的写入频率。每执行多少次事务写入一次(这个参数性能消耗很大，但可减小MySQL崩溃造成的损失)
sync_binlog = 1
#这个参数一般用在主主同步中，用来错开自增值, 防止键值冲突
auto_increment_offset = 1
#这个参数一般用在主主同步中，用来错开自增值, 防止键值冲突
auto_increment_increment = 1
#二进制日志自动删除的天数，默认值为0,表示“没有自动删除”，启动时和二进制日志循环时可能删除
expire_logs_days = 7
#将函数复制到slave
log_bin_trust_function_creators = 1
```

3. 重启 Mysql，创建允许从服务器同步数据的账户

```bash
#创建slave账号account，密码123456
mysql>grant replication slave on *.* to 'account'@'10.10.20.116' identified by '123456';
#更新数据库权限
mysql>flush privileges;
```

4. 查看主服务器状态

```bash
mysql>show master status\G;
***************** 1. row ****************
            File: mysql-bin.000033 #当前记录的日志
        Position: 337523 #日志中记录的位置
    Binlog_Do_DB:
Binlog_Ignore_DB:
```

---

# 从服务器配置

1. 修改 mysql 配置

```bash
vi /etc/msyql/msyql.conf.d/msyqld.conf
```

2. 在[mysqld]中添加

```bash
server_id = 2
log-bin = mysql-bin
log-slave-updates
sync_binlog = 0
#log buffer将每秒一次地写入log file中，并且log file的flush(刷到磁盘)操作同时进行。该模式下在事务提交的时候，不会主动触发写入磁盘的操作
innodb_flush_log_at_trx_commit = 0
#指定slave要复制哪个库
binlog-do-db = db
#MySQL主从复制的时候，当Master和Slave之间的网络中断，但是Master和Slave无法察觉的情况下（比如防火墙或者路由问题）。Slave会等待slave_net_timeout设置的秒数后，才能认为网络出现故障，然后才会重连并且追赶这段时间主库的数据
slave-net-timeout = 60
log_bin_trust_function_creators = 1
```

3. 执行同步命令

```bash
#执行同步命令，设置主服务器ip，同步账号密码，同步位置
mysql>change master to master_host='10.10.20.111',master_user='account',master_password='123456',master_log_file='mysql-bin.000033',master_log_pos=337523;
#开启同步功能
mysql>start slave;
```

4. 查看从服务器状态

```bash
mysql>show slave status\G;
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.10.20.111
                  Master_User: account
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000033
          Read_Master_Log_Pos: 337523
               Relay_Log_File: db2-relay-bin.000002
                Relay_Log_Pos: 337686
        Relay_Master_Log_File: mysql-bin.000033
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
          ...
```

5. 查看主服务器中的 slave

```bash
>show slave hosts
```

---

# 部署中的坑

1. 因为 1146 错误导致无法同步，在配置文件中添加(slave_skip_errors=1146)

