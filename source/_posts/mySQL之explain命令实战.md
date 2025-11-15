---
title: mySQL之explain命令实战
date: 2025-11-15 19:11:25
tags: mysql
categories: 数据库
---

# 1. 实战

1. SQL

```sql
explain  SELECT m.id, m.category, m.model_name, m.api_host, m.model_type, m.icon_name
FROM jai_brain.chat_model m
WHERE m.model_show = 0
  AND (
    m.visibility = 0
        OR (
        m.visibility = 1
            AND EXISTS (
            SELECT 1
            FROM jai_brain.chat_model_resource r
                     JOIN jai_brain.role_chat_model_resource p ON p.resource_id = r.id
                     JOIN sys_user_role ur ON ur.role_id = p.role_id
            WHERE r.chat_model_id = m.id
              AND r.resource_type = 0
              AND ur.user_id = '1945392421362470914'
        )
        )
    )
ORDER BY m.order_num;
```





加索引之前：

![img](https://panyuro.oss-cn-beijing.aliyuncs.com/1761705397664-07287dc4-fa82-4331-9a41-118036347b8d.png)

![img](https://panyuro.oss-cn-beijing.aliyuncs.com/1761705151416-cabe2353-1963-4fa8-b196-9c68120fd7a7.png)

![img](https://panyuro.oss-cn-beijing.aliyuncs.com/1761705171929-89689938-6c97-40a3-ba30-006214e8ff98.png)

[2025-10-29 10:37:46] 4 rows retrieved starting from 1 in 393 ms (execution: 40 ms, fetching: 353 ms)



执行加索引

```sql
create unique index role_id_resource_id_uindex
    on role_chat_model_resource (role_id, resource_id);

create unique index chat_model_id_resource_type_permission_key_uindex
    on chat_model_resource (chat_model_id, resource_type, permission_key);
```





加索引后：

![img](https://panyuro.oss-cn-beijing.aliyuncs.com/1761705022123-0ff452b6-bb03-45ad-8b61-475c14d92fb4.png)![img](https://panyuro.oss-cn-beijing.aliyuncs.com/1761705037061-1f094acb-ea16-4a34-958f-460f11d8f216.png)

![img](https://panyuro.oss-cn-beijing.aliyuncs.com/1761705945650-a559a98c-cfba-4233-bfa2-f61b81239af2.png)

[2025-10-29 10:39:30] 4 rows retrieved starting from 1 in 403 ms (execution: 17 ms, fetching: 386 ms)



# 2. 如何阅读 EXPLAIN（步骤化）

运行：

```plain
EXPLAIN SELECT ...;
```

看输出的关键列（按优先级）：

1. `id` / `select_type`
   - 标识查询的层次（主查询 / 子查询 / derived 等）。
   - 如果看到 `DEPENDENT SUBQUERY`，说明子查询会对外层每行执行一次 —— 很慢。
2. `table`
   - 当前操作的表名或临时/derived 表。
3. `type` （非常重要）
   - 代表访问方式（从好到差）：`system``const``eq_ref``ref``range``index``ALL`。
   - `ALL` = 全表扫描（坏），`ref`/`range`/`eq_ref` 表示使用索引（好）。
4. `possible_keys` / `key` / `key_len`
   - `possible_keys`：优化器可选的索引。
   - `key`：实际用到的索引（若为空则未用索引）。
   - `key_len`：使用了索引的长度（越多列越长，通常越好）。
5. `rows`
   - 优化器估算需读取的行数（估算值，不是实际）。大值表示会扫描大量数据。
6. `Extra`（非常关键的可读信号）
   - `Using where`：使用 WHERE 筛选（正常）。
   - `Using index`：覆盖索引（好，避免回表）。
   - `Using filesort`：需要额外排序（可能慢，若可通过索引避免更好）。
   - `Using temporary`：使用临时表（GROUP BY / ORDER BY 等导致，注意）。
   - `Using join buffer` 等也提示连接方式问题。

## 读法步骤（实际做法）

1. 看主查询那行（`id` 最小）：
   - `type` 如果是 `ALL` → 寻找能把它变成 `ref/range` 的索引（看 WHERE、JOIN、ORDER BY 的列）。
   - `Extra` 若出现 `Using filesort` → 考虑给 `ORDER BY` 的列建合适复合索引（按 WHERE 列次序 + order 列）。
2. 看子查询（若有）：
   - `DEPENDENT SUBQUERY` 很糟 → 考虑改写为 `IN (subquery)` 或先取 role_id 再 join 小表。
3. 看 `possible_keys` 与 `key`：
   - 若 `possible_keys` 有值但 `key` 为空，说明有索引但没被选中，可能是列顺序或函数使用导致。
4. 看 `rows`：若某一步 `rows` 值很大（百万级），那就是瓶颈所在表。
5. `EXPLAIN FORMAT=JSON` 给更多细节（cost、filtered、attached_condition），适合复杂查询。
