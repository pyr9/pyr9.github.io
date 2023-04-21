---
title: postgresql 常用SQL整理
date: 2023-04-21 10:53:27
tags:
categories:
---

1.  删除id最小的重复行

```sql
DELETE FROM mes.stool_po where id in(
    SELECT MIN(id) FROM mes.stool_po GROUP BY purchasing_doc_number, purchasing_doc_item_number HAVING COUNT(id) > 1
)
```

2. 查询根据purchasing_doc_number, purchasing_doc_item_number分组后，重复的数据id

```sql
SELECT array_agg(id) FROM mes.stool_po GROUP BY purchasing_doc_number, purchasing_doc_item_number HAVING COUNT(id) > 1;
```



