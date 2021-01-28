# pg_rman备份postgresql


# pg_rman 工具

pg_rman 是 PostgreSQL 的备份与恢复工具，支持全量、增量、归档三种备方式。

# pg_rman 安装

pg_rman 没有 deb 包,需要编译安装。

## 开源使用文档

- http://ossc-db.github.io/pg_rman/index.html

## 下载链接

- https://github.com/ossc-db/pg_rman/releases

## 编译安装

- 解压压缩包后,在安装目录下 make && make install

## 安装注意事项

- 一定要下载和本机 pgsql 版本相同的压缩包编译安装
- 依赖(libpq-dev\postgresql-server-dev-all\libpam0g-dev\libedit-dev\libselinux1-dev)
- pg_rman 默认安装在 postgresql 的 bin 目录下

# pg_rman 使用

## 使用前注意事项

- 新建 BACKUP_PATH\SRVLOG_PATH\ARCLOG_PATH(归档日志目录)并设置主:组为 postgres
- pg_rman 需要加入环境变量

```bash
export PGDATA=/var/lib/postgresql/9.5/main
export PATH=/usr/lib/postgresql/9.5/bin:$PATH
export ARCLOG_PATH=/tmp/postgres/arc_log
export SRVLOG_PATH=/tmp/postgres/pg_log
export BACKUP_PATH=/tmp/postgres/backup
```

- 配置 postgresql.conf 开启日志归档,配置完成后重启 postgresql

```bash
wal_level = archive
archive_mode = on
archive_command = 'cp %p /tmp/postgres/arc_log/%f'
archive_timeout = 60
```

- 初始化备份目录

```bash
pg_rman init -B /tmp/postgres/backup
```

- su postgres

```
进入postgres用户下执行备份操作，若使用其他用户备份会导致
备份恢复后权限不对无法启动postgresql
```

# 使用 pg_rman 备份

## 全量备份

```bash
postgres@C01U16BOX02:~$ pg_rman backup -b full
INFO: copying database files
INFO: copying archived WAL files
INFO: backup complete
INFO: Please execute 'pg_rman validate' to verify the files are correctly copied.
postgres@C01U16BOX02:~$ pg_rman validate
INFO: validate: "2019-07-03 15:52:08" backup and archive log files by CRC
INFO: backup "2019-07-03 15:52:08" is valid
```

## 增量备份

```bash
postgres@C01U16BOX02:~$ pg_rman backup -b incremental
INFO: copying database files
INFO: copying archived WAL files
INFO: backup complete
INFO: Please execute 'pg_rman validate' to verify the files are correctly copied.
```

## 备份校验

```bash
每次备份结束必须做一次校验，否则备份不能用来恢复，增量备份时也不会用它来做增量比较
postgres@C01U16BOX02:~$ pg_rman validate
INFO: validate: "2019-07-03 15:52:08" backup and archive log files by CRC
INFO: backup "2019-07-03 15:52:08" is valid
```

## 查看备份

```bash
postgres@C01U16BOX02:~$ pg_rman show
=====================================================================
StartTime           EndTime              Mode    Size   TLI  Status
=====================================================================
2019-07-03 15:52:08  2019-07-03 15:52:28  FULL    86MB     1  OK
2019-07-03 15:51:02  2019-07-03 15:51:05  INCR    45MB     1  OK
2019-07-03 15:49:58  2019-07-03 15:50:00  INCR    61MB     1  OK
2019-07-03 15:46:13  2019-07-03 15:46:17  FULL    68MB     1  OK
```

# 使用 pg_rman 进行恢复

## 注意事项

- 恢复数据必须实在 postgresql 停止的状态下
- 恢复前请使用 pg_rman show 查看所需要恢复的时间点

## 恢复操作

```bash
pg_rman restore [options:] #不指定options则默认恢复到最近一次全量备份
--recovery-target-time #指定恢复时间点
postgres@C01U16BOX02:~$ pg_rman restore --recovery-target-time "2019-07-03 15:51:02"
INFO: the recovery target timeline ID is not given
INFO: use timeline ID of current database cluster as recovery target: 1
INFO: calculating timeline branches to be used to recovery target point
INFO: searching latest full backup which can be used as restore start point
INFO: found the full backup can be used as base in recovery: "2019-07-03 15:46:13"
INFO: copying online WAL files and server log files
INFO: clearing restore destination
INFO: validate: "2019-07-03 15:46:13" backup and archive log files by SIZE
INFO: backup "2019-07-03 15:46:13" is valid
INFO: restoring database files from the full mode backup "2019-07-03 15:46:13"
INFO: searching incremental backup to be restored
INFO: validate: "2019-07-03 15:49:58" backup and archive log files by SIZE
INFO: backup "2019-07-03 15:49:58" is valid
INFO: restoring database files from the incremental mode backup "2019-07-03 15:49:58"
INFO: searching backup which contained archiv
```

## 备注

- 详细文档可见 github

