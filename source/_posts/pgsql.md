---
title: pgsql
date: 2024-07-03 22:48:10
tags:
categories:
---

# 1. pgsql的优势

### 1. **开源和社区支持**

- **开源**：PostgreSQL 是完全开源的，无需支付许可费用。
- **活跃的社区**：有一个活跃的社区提供支持和开发，不断改进和更新功能。

### 2. **强大的功能集**

- **ACID 合规**：完全遵循 ACID 原则，确保数据的可靠性和一致性。
- **高级 SQL 功能**：支持复杂查询、子查询、窗口函数、CTE（Common Table Expressions）等。
- **并发控制**：使用多版本并发控制（MVCC）实现高效并发处理，减少锁争用。

### 3. **扩展性和可扩展性**

- **扩展支持**：支持多种扩展，如 PostGIS（地理信息系统扩展）、Full Text Search（全文搜索扩展）。
- **可编程性**：支持存储过程和触发器，可以使用多种语言编写，如 PL/pgSQL、PL/Python、PL/Perl。

### 4. **数据完整性**

- **数据类型丰富**：支持丰富的数据类型，包括 JSON、数组、范围、枚举、几何数据等。
- **约束和触发器**：支持多种约束（如主键、外键、唯一约束）和触发器，确保数据完整性。
- **外部数据包装器**：通过 Foreign Data Wrapper（FDW）可以查询外部数据源。

### 5. **高性能和优化**

- **并行查询**：支持并行查询，充分利用多核 CPU 提高查询性能。
- **高级索引**：支持多种索引类型（如 B-tree、GIN、GiST、BRIN），满足不同查询需求。
- **内存优化**：支持表和索引的内存缓存，提高访问速度。

### 6. **高可用性和容灾**

- **流复制**：支持主从复制（同步和异步），实现高可用性。
- **逻辑复制**：支持逻辑复制，可以进行部分数据的复制和定制化的数据同步。
- **备份和恢复**：支持热备份和 PITR（时间点恢复），确保数据安全。

### 7. **灵活性和兼容性**

- **跨平台支持**：支持多种操作系统，如 Linux、Windows、macOS。
- **标准兼容**：高度兼容 SQL 标准，易于迁移和集成。
- **扩展支持**：通过插件和扩展（如 pg_partman、pg_stat_statements）可以扩展功能。

### 8. **管理和监控工具**

- **pgAdmin**：强大的图形化管理工具，便于管理和监控数据库。
- **内置统计功能**：提供丰富的统计信息和视图，便于监控和优化数据库性能。

### 9. **安全性**

- **访问控制**：支持基于角色和用户的访问控制，提供细粒度的权限管理。
- **加密**：支持数据传输加密和磁盘加密，确保数据安全。
- **认证方式**：支持多种认证方式，如密码认证、Kerberos、LDAP 等。

# 2. pgsql数据类型

### 1. 数值类型

- **整数类型**
  - `smallint`：2字节整数，范围 -32,768 到 32,767
  - `integer`：4字节整数，范围 -2,147,483,648 到 2,147,483,647
  - `bigint`：8字节整数，范围 -9,223,372,036,854,775,808 到 9,223,372,036,854,775,807
- **小数类型**
  - `decimal` 或 `numeric`：可变精度，适用于需要精确小数的场合
  - `real`：4字节浮点数
  - `double precision`：8字节浮点数
- **序列/自动增长**
  - `serial`：自动递增的4字节整数
  - `bigserial`：自动递增的8字节整数

### 2. 字符串类型

- **定长字符串**
  - `char(n)`：固定长度字符类型，长度为 n
- **变长字符串**
  - `varchar(n)`：可变长度字符类型，最大长度为 n
  - `text`：可变长度字符类型，没有长度限制

### 3. 日期和时间类型

- 日期和时间
  - `date`：日期类型，范围为 4713 BC 到 5874897 AD
  - `time`：无时区的时间类型
  - `timestamp`：无时区的时间戳类型
  - `timestamptz`：带时区的时间戳类型
  - `interval`：时间间隔

### 4. 布尔类型

- 布尔值
  - `boolean`：值为 `TRUE`、`FALSE` 或 `NULL`

### 5. 枚举类型

- 枚举
  - `enum`：定义一组固定值

### 6. 数组类型

- 数组
  - 任意类型的数组，如 `integer[]`、`text[]`

### 7. JSON 类型

- JSON 数据
  - `json`：存储JSON数据
  - `jsonb`：存储二进制JSON数据，支持索引和更快的查询

### 8. 范围类型

- 范围
  - `int4range`、`int8range`、`numrange`、`tsrange`、`tstzrange`、`daterange`：表示范围值

### 9. 几何类型

- 几何数据
  - `point`、`line`、`lseg`、`box`、`path`、`polygon`、`circle`：用于几何计算的类型

### 10. 网络地址类型

- 网络地址
  - `cidr`：IPv4或IPv6网络
  - `inet`：IPv4或IPv6地址
  - `macaddr`：MAC地址

### 11. UUID 类型

- UUID
  - `uuid`：通用唯一标识符

### 12. Hstore 类型

- 键值对存储
  - `hstore`：存储键值对数据

### 13. 复合类型

- 复合
  - 用户定义的复合类型，包含多个字段

# 3. pgsql索引类型

### 1. B-tree 索引

- **特点**：默认索引类型，适用于大多数情况。

- **用途**：适用于等值查询、范围查询、排序。

- 示例

  ```
  CREATE INDEX idx_example_btree ON example_table (column_name);
  ```

### 2. Hash 索引

- **特点**：适用于等值查询，不支持范围查询。

- **用途**：仅适用于等值比较，如 `=` 和 `IN`。

- 示例

  ```
  CREATE INDEX idx_example_hash ON example_table USING hash (column_name);
  ```

### 3. GIN（Generalized Inverted Index）

- **特点**：适用于包含复杂结构数据的列，如数组、全文搜索。

- **用途**：支持多值包含查询，如数组、JSONB、全文搜索。

- 示例

  ```
  CREATE INDEX idx_example_gin ON example_table USING gin (column_name);
  ```

### 4. GiST（Generalized Search Tree）

- **特点**：适用于需要自定义索引行为的场景。

- **用途**：支持几何数据、全文搜索、范围查询、KNN查询等。

- 示例

  ```
  CREATE INDEX idx_example_gist ON example_table USING gist (column_name);
  ```

### 5. SP-GiST（Space-Partitioned Generalized Search Tree）

- **特点**：适用于空间数据和稀疏数据。

- **用途**：支持空间数据、高维数据等。

- 示例

  ```
  CREATE INDEX idx_example_spgist ON example_table USING spgist (column_name);
  ```

### 6. BRIN（Block Range INdexes）

- **特点**：适用于非常大的表，存储空间小，适合顺序扫描。

- **用途**：适用于顺序访问的大型表。

- 示例

  ```
  CREATE INDEX idx_example_brin ON example_table USING brin (column_name);
  ```

### 7. Full Text Search 索引

- **特点**：适用于全文搜索。

- **用途**：为文本数据创建高效的全文搜索索引。

- 示例

  ```
  CREATE INDEX idx_example_tsvector ON example_table USING gin (to_tsvector('english', column_name));
  ```

### 8. Composite 索引

- **特点**：包含多个列的组合索引。

- **用途**：优化涉及多个列的查询。

- 示例

  ```
  CREATE INDEX idx_example_composite ON example_table (column1, column2);
  ```

### 9. Partial 索引

- **特点**：基于条件的部分索引。

- **用途**：只索引满足特定条件的行，减少索引大小。

- 示例

  ```
  CREATE INDEX idx_example_partial ON example_table (column_name) WHERE condition;
  ```

### 10. Unique 索引

- **特点**：确保列值唯一。

- **用途**：保证列值唯一性，自动创建在主键和唯一约束上。

- 示例

  ```
  CREATE UNIQUE INDEX idx_example_unique ON example_table (column_name);
  ```

### 11. Expression 索引

- **特点**：基于表达式的索引。

- **用途**：对计算列或表达式创建索引。

- 示例

  ```
  CREATE INDEX idx_example_expression ON example_table ((column1 + column2));
  ```
