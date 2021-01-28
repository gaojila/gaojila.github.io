# greenplum并行备份与恢复


# 备份工具选择

## gpcrondump

### 说明

gpcrondump 实用程序将数据库的内容转储为 SQL 脚本文件，然后可以使用它们在以后使用它来还原数据库架构和用户数据 gpdbrestore。在转储操作期间，用户仍将具有对数据库的完全访问权限。

### 缺点

```
Backing up a database with gpcrondump while simultaneously running ALTER TABLE might cause gpcrondump to fail.
Backing up a database with gpcrondump while simultaneouslyrunning DDL commands might cause issues with locks.
You might see either the DDL command or gpcrondump waiting to acquire locks.
大概就是运行的时候更改表会备份失败,运行 DDL 会产生锁表问题
```

## gpbackup

### 说明

将数据库的内容备份到元数据文件和数据文件的集合中，这些文件和数据文件可用于以后使用它来还原数据库 gprestore。 默认情况下， gpbackup 备份指定数据库中的对象以及全局 Greenplum 数据库系统对象。您可以选择提供-globals 选项 gprestore 恢复全局对象.
gpbackup 默认情况下，将对象元数据文件和 DDL 文件存储在 Greenplum 数据库主数据目录中。Greenplum 数据库细分使用复制..关于部分命令将备份表的数据存储在位于每个段的数据目录中的压缩 CSV 数据文件中。

### 安装

- gpbackup 由 go 编写，可一次编译到处执行。
- [源码编译安装](https://github.com/greenplum-db/gpbackup)

### 参数

```
-dbname database_name
    需要。指定要备份的数据库。
-backupdir 目录
    可选的。将所有必需的备份文件（元数据文件和数据文件）复制到指定的目录。您必须将目录指定为绝对路径（不是相对路径）。如果您不提供此选项，则会在$ MASTER_DATA_DIRECTORY / backups / YYYYMMDD / YYYYMMDDhhmmss / 目录中的Greenplum Database主机上创建元数据文件 。段主机在<seg_dir> / backups / YYYYMMDD / YYYYMMDDhhmmss /目录中创建CSV数据文件 。指定自定义备份目录时，会将文件复制到备份目录的子目录中的这些路径。
-data-only
    可选的。仅将表数据备份到CSV文件中，但不备份重新创建表和其他数据库对象所需的元数据文件。
-debug
    可选的。在操作期间显示详细的调试消息。
-exclude-schema schema_name
    可选的。指定要从备份中排除的数据库模式。您可以多次指定此选项以排除多个模式。您无法将此选项与-include-模式选项。有关详细信息，请参阅筛选备份的内容。
-exclude-table-file file_name
    可选的。指定包含要从备份中排除的表列表的文本文件。文本文件中的每一行都必须使用该格式定义一个表 <模式名称> <表名>。该文件不得包含尾随行。如果表或模式名称使用小写字母，数字或下划线字符以外的任何字符，则必须在双引号中包含该名称。
您不能结合使用此选项 -leaf分区数据。虽然您可以在指定的文件中指定叶子分区名称 -exclude表文件， gpbackup 忽略分区名称。
    有关 详细信息，请参阅筛选备份的内容。
-include-schema schema_name
    可选的。指定要包括在备份中的数据库模式。您可以多次指定此选项以包含多个模式。如果指定此选项，则后续未包含的任何模式-include-模式备份集中省略了选项。您无法将此选项与 -exclude-模式选项。有关详细信息，请参阅筛选备份的内容。
-include-table-file file_name
    可选的。指定包含要包括在备份中的表列表的文本文件。文本文件中的每一行都必须使用该格式定义一个表 <模式名称> <表名>。该文件不得包含尾随行。如果表或模式名称使用小写字母，数字或下划线字符以外的任何字符，则必须在双引号中包含该名称。备份集中将省略此文件中未列出的任何表。
您可以选择指定表叶子分区名称来代替表名称，以便在备份中仅包含特定叶子分区 -leaf分区数据 选项。
有关 详细信息，请参阅筛选备份的内容。
--leaf-partition-data
    可选的。对于分区表，为每个叶子分区创建一个数据文件，而不是为整个表创建一个数据文件（默认值）。使用此选项还可以指定要包含在备份中的单个叶子分区 -include表文件选项。您不能结合使用此选项-exclude表文件。
--metadata-only
    可选的。仅创建重新创建数据库对象所需的元数据文件（DDL），但不备份实际的表数据。
--no-compression
    可选的。不要压缩表数据CSV文件。
--quiet
    可选的。禁止所有非警告，非错误日志消息。
-verbose
    可选的。打印详细日志消息。
--version
    可选的。打印版本号并退出。
--with-stats
    可选的。在备份集中包含查询计划统计信息。
```

# gpbackup 备份实例说明

## 完全备份

```bash
gpadmin@gpmaster:~$ gpbackup --dbname test --backup-dir /var/tmp/0715 --leaf-partition-data
20190716:10:25:59 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Starting backup of database test
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Backup Timestamp = 20190716102559
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Backup Database = test
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Gathering table state information
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Acquiring ACCESS SHARE locks on tables
Locks acquired:  1 / 1 [============================================================] 100.00% 0s
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Gathering additional table metadata
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Metadata will be written to /var/tmp/0715/gpseg-1/backups/20190716/20190716102559/gpbackup_20190716102559_metadata.sql
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Writing global database metadata
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Metadata will be written to /var/tmp/0715/gpseg-1/backups/20190716/20190716102559/gpbackup_20190716102559_metadata.sql
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Writing global database metadata
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Global database metadata backup complete
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Writing pre-data metadata
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Pre-data metadata backup complete
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Writing post-data metadata
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Post-data metadata backup complete
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Writing data to file
Tables backed up:  1 / 1 [==========================================================] 100.00% 0s
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Data backup complete
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Writing data to file
Tables backed up:  1 / 1 [==========================================================] 100.00% 0s
20190716:10:26:00 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Data backup complete
20190716:10:26:01 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Found neither /opt/gpdb/bin/gp_email_contacts.yaml nor /home/gpadmin/gp_email_contacts.yaml
20190716:10:26:01 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Email containing gpbackup report /var/tmp/0715/gpseg-1/backups/20190716/20190716102559/gpbackup_20190716102559_report will not be sent
20190716:10:26:01 gpbackup:gpadmin:gpmaster:003082-[INFO]:-Backup completed successfully
```

## 增量备份

```bash
gpadmin@gpmaster:~$ gpbackup --dbname test --backup-dir /var/tmp/0715 --leaf-partition-data --incremental
20190716:10:27:08 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Starting backup of database test
20190716:10:27:08 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Backup Timestamp = 20190716102708
20190716:10:27:08 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Backup Database = test
20190716:10:27:08 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Gathering table state information
20190716:10:27:08 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Acquiring ACCESS SHARE locks on tables
Locks acquired:  2 / 2 [============================================================] 100.00% 0s
20190716:10:27:08 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Gathering additional table metadata
20190716:10:27:08 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Metadata will be written to /var/tmp/0715/gpseg-1/backups/20190716/20190716102708/gpbackup_20190716102708_metadata.sql
20190716:10:27:08 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Writing global database metadata
20190716:10:27:08 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Global database metadata backup complete
20190716:10:27:08 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Writing pre-data metadata
20190716:10:27:09 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Pre-data metadata backup complete
20190716:10:27:09 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Writing post-data metadata
20190716:10:27:09 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Post-data metadata backup complete
20190716:10:27:09 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Basing incremental backup off of backup with timestamp = 20190716102559
20190716:10:27:09 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Writing data to file
Tables backed up:  2 / 2 [==========================================================] 100.00% 0s
20190716:10:27:09 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Data backup complete
20190716:10:27:10 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Found neither /opt/gpdb/bin/gp_email_contacts.yaml nor /home/gpadmin/gp_email_contacts.yaml
20190716:10:27:10 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Email containing gpbackup report /var/tmp/0715/gpseg-1/backups/20190716/20190716102708/gpbackup_20190716102708_report will not be sent
20190716:10:27:10 gpbackup:gpadmin:gpmaster:003120-[INFO]:-Backup completed successfully
```

## 表级别备份

```bash
gpadmin@gsdw_dev-master:~$ gpbackup --dbname gsdw --backup-dir /data/gpbackup --include-table dw_agg.agg_cus_fct_category_for_mkt
20190801:14:51:46 gpbackup:gpadmin:gsdw_dev-master:016502-[INFO]:-Starting backup of database gsdw
20190801:14:51:46 gpbackup:gpadmin:gsdw_dev-master:016502-[INFO]:-Backup Timestamp = 20190801145145
20190801:14:51:46 gpbackup:gpadmin:gsdw_dev-master:016502-[INFO]:-Backup Database = gsdw
20190801:14:51:46 gpbackup:gpadmin:gsdw_dev-master:016502-[INFO]:-Gathering table state information
20190801:14:51:46 gpbackup:gpadmin:gsdw_dev-master:016502-[INFO]:-Acquiring ACCESS SHARE locks on tables
Locks acquired:  1 / 1 [==================================================] 100.00% 0s
20190801:14:51:46 gpbackup:gpadmin:gsdw_dev-master:016502-[INFO]:-Gathering additional table metadata
20190801:14:51:46 gpbackup:gpadmin:gsdw_dev-master:016502-[INFO]:-Metadata will be written to /data/gpbackup/gpseg-1/backups/20190801/20190801145145/gpbackup_20190801145145_metadata.sql
20190801:14:51:46 gpbackup:gpadmin:gsdw_dev-master:016502-[INFO]:-Writing pre-data metadata
20190801:14:51:46 gpbackup:gpadmin:gsdw_dev-master:016502-[INFO]:-Pre-data metadata backup complete
20190801:14:51:46 gpbackup:gpadmin:gsdw_dev-master:016502-[INFO]:-Writing post-data metadata
20190801:14:51:47 gpbackup:gpadmin:gsdw_dev-master:016502-[INFO]:-Post-data metadata backup complete
20190801:14:51:47 gpbackup:gpadmin:gsdw_dev-master:016502-[INFO]:-Writing data to file
Tables backed up:  1 / 1 [================================================] 100.00% 0s
20190801:14:51:47 gpbackup:gpadmin:gsdw_dev-master:016502-[INFO]:-Data backup complete
20190801:14:51:48 gpbackup:gpadmin:gsdw_dev-master:016502-[INFO]:-Found neither /opt/gpdb/bin/gp_email_contacts.yaml nor /home/gpadmin/gp_email_contacts.yaml
20190801:14:51:48 gpbackup:gpadmin:gsdw_dev-master:016502-[INFO]:-Email containing gpbackup report /data/gpbackup/gpseg-1/backups/20190801/20190801145145/gpbackup_20190801145145_report will not be sent
20190801:14:51:48 gpbackup:gpadmin:gsdw_dev-master:016502-[INFO]:-Backup completed successfully
```

## 备份恢复

```bash
gpadmin@gpmaster:~$ gprestore -backup-dir /var/tmp/0715 -timestamp 20190715101501 --create-db
20190715:10:16:17 gprestore:gpadmin:gpmaster:002765-[INFO]:-Restore Key = 20190715101501
20190715:10:16:17 gprestore:gpadmin:gpmaster:002765-[INFO]:-Restoring pre-data metadata
Pre-data objects restored:  8 / 8 [=================================================] 100.00% 0s
20190715:10:16:18 gprestore:gpadmin:gpmaster:002765-[INFO]:-Pre-data metadata restore complete
Tables restored:  2 / 2 [===========================================================] 100.00% 0s
20190715:10:16:18 gprestore:gpadmin:gpmaster:002765-[INFO]:-Data restore complete
20190715:10:16:18 gprestore:gpadmin:gpmaster:002765-[INFO]:-Restoring post-data metadata
Post-data objects restored:  2 / 2 [================================================] 100.00% 0s
20190715:10:16:18 gprestore:gpadmin:gpmaster:002765-[INFO]:-Post-data metadata restore complete
20190715:10:16:18 gprestore:gpadmin:gpmaster:002765-[INFO]:-Found neither /opt/gpdb/bin/gp_email_contacts.yaml nor /home/gpadmin/gp_email_contacts.yaml
20190715:10:16:18 gprestore:gpadmin:gpmaster:002765-[INFO]:-Email containing gprestore report /var/tmp/0715/gpseg-1/backups/20190715/20190715101501/gprestore_20190715101501_20190715101617_report will not be sent
20190715:10:16:18 gprestore:gpadmin:gpmaster:002765-[INFO]:-Restore completed successfully
```

## 表级别恢复

```bash
gpadmin@gsdw_dev-master:~$ gprestore --backup-dir /var/tmp/0724 --timestamp 20190724132252 --include-table test.test_user_tb
20190724:14:32:03 gprestore:gpadmin:gsdw_dev-master:001423-[INFO]:-Restore Key = 20190724132252
20190724:14:32:04 gprestore:gpadmin:gsdw_dev-master:001423-[INFO]:-Restoring pre-data metadata
Pre-data objects restored:  0 / 11 [-------------------------------------------------------]   0.00%20190724:14:32:05 gprestore:gpadmin:gsdw_dev-master:001423-[WARNING]:-Schema meta_data already exists
20190724:14:32:05 gprestore:gpadmin:gsdw_dev-master:001423-[WARNING]:-Schema test already exists
Pre-data objects restored:  11 / 11 [===================================================] 100.00% 1s
20190724:14:32:07 gprestore:gpadmin:gsdw_dev-master:001423-[INFO]:-Pre-data metadata restore complete
Tables restored:  1 / 1 [===============================================================] 100.00% 0s
20190724:14:32:09 gprestore:gpadmin:gsdw_dev-master:001423-[INFO]:-Data restore complete
20190724:14:32:09
```

## 备份恢复前后比较

- 备份前表结构
  ![](https://raw.githubusercontent.com/gaojila/images/master/greenplum%E5%B9%B6%E8%A1%8C%E5%A4%87%E4%BB%BD%E4%B8%8E%E6%81%A2%E5%A4%8D/2019-08-01-14-59-22.png)
- 备份前数据量
  ![](https://raw.githubusercontent.com/gaojila/images/master/greenplum%E5%B9%B6%E8%A1%8C%E5%A4%87%E4%BB%BD%E4%B8%8E%E6%81%A2%E5%A4%8D/2019-08-01-15-01-43.png)
- 删除 cn_accounting_ar_invoice_lines 表做恢复演示
  ![](https://raw.githubusercontent.com/gaojila/images/master/greenplum%E5%B9%B6%E8%A1%8C%E5%A4%87%E4%BB%BD%E4%B8%8E%E6%81%A2%E5%A4%8D/2019-08-01-15-16-10.png)
- 备份恢复后表结构
  ![](https://raw.githubusercontent.com/gaojila/images/master/greenplum%E5%B9%B6%E8%A1%8C%E5%A4%87%E4%BB%BD%E4%B8%8E%E6%81%A2%E5%A4%8D/2019-08-01-15-21-17.png)
- 备份恢复后数据量
  ![](https://raw.githubusercontent.com/gaojila/images/master/greenplum%E5%B9%B6%E8%A1%8C%E5%A4%87%E4%BB%BD%E4%B8%8E%E6%81%A2%E5%A4%8D/2019-08-01-15-22-55.png)

# 注意事项（坑）

- gpbackup 需要下载放到 greenplum 的环境变量中去
- gprestore 恢复数据的前提 greenplum 有这个要恢复的数据库且数据库要是空的..........
- gprestore 恢复表的前提 greenplum 要恢复的表是空的..........
- 备份和和恢复最好指定备份路径,不然恢复的时候找不到.........
- 如果要使用增量备份,在全备份的时候需要加（ --leaf-partition-data)参数............
- [详细说明请见官方文档](https://gpdb.docs.pivotal.io/43320/admin_guide/managing/backup-gpbackup.html)

