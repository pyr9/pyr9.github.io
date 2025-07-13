---
title: oracle数据迁移
date: 2025-07-13 11:10:33
tags:
categories: 数据库
---

# 1. oracle数据库迁移

## 1. 导出dmp文件

```shell
expdp mycim3_qj_new/mycim3_qj_new@//10.108.140.102:1521/ORCL schemas=MYCIM3_QJ_NEW directory=DATA_PUMP_DIR dumpfile=export.dmp logfile=export.log
```

注：导出结束以后会显示这个文件在哪个路径下面，注意看最后几行

## 2. 创建文件夹，并上传dmp文件

```shell
mkdir -p /home/oracle/import
```

## 3. 执行导入

```shell
sqlplus sys/WElcome_123_@//172.28.46.12:1521/mestest as sysdba
```

1. 创建新用户，并将文件夹权限赋给他

```shell
CREATE USER mycim3_qj_new IDENTIFIED BY "mycim3_qj_new";
GRANT CREATE SESSION TO mycim3_qj_new;
GRANT DATAPUMP_IMP_FULL_DATABASE TO mycim3_qj_new;
ALTER USER MYCIM3_QJ_NEW QUOTA UNLIMITED ON users; 

CREATE DIRECTORY import_dir AS '/home/oracle/import';
GRANT READ, WRITE ON DIRECTORY import_dir TO mycim3_qj_new;
```

2. 创建TABLESPACE

```shell
CREATE TABLESPACE TBS_MYCIM
DATAFILE '/u01/app/oracle/oradata/MESTEST/tbs_mycim_01.dbf'
SIZE 1024M
AUTOEXTEND ON NEXT 10M MAXSIZE UNLIMITED;

CREATE TABLESPACE TBS_MYCIM_PARTTABLE
DATAFILE '/u01/app/oracle/oradata/MESTEST/tbs_mycim_parttable_01.dbf'
SIZE 1024M
AUTOEXTEND ON NEXT 10M MAXSIZE UNLIMITED;
  
```

3. 执行导入

```shell
impdp mycim3_qj_new/mycim3_qj_new@//172.28.49.12:1521/mestest schemas=MYCIM3_QJ_NEW directory=import_dir dumpfile=2025.4.28EXPORT.DMP logfile=import.log
```

# 2.  为什么 Oracle 不直接支持导出为 SQL？

1. **效率问题**：生成包含所有数据的SQL语句文件会非常庞大，尤其是在处理大量数据时，这样的文件不仅占用更多存储空间，而且在传输和加载过程中也会消耗更多时间。
2. **复杂性管理**：Oracle 支持的数据类型远比 MySQL 复杂，例如 BFILE, XMLType 等特殊类型，直接转换为 SQL 语句可能会遇到诸多挑战。
3. **事务支持**：Oracle 强调事务的一致性和隔离级别，在导出过程中保证数据一致性对于企业级应用至关重要，而二进制格式在这方面提供了更好的保障。
4. **高级特性**：像分区表、外部表等高级特性在转换为 SQL 时可能无法完全保留其原始属性，而二进制格式则能更好地支持这些特性。
