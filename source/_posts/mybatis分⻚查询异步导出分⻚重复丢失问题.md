---
title: mybatis分⻚查询异步导出分⻚重复/丢失问题
date: 2025-06-03 21:44:49
tags:
categories:
---

# 1. 初始状态

我们有一个库存表 inventory，里面有 25 条记录，按 id 排序（从小到大）。

每页显示 10条记录：

第1页：LIMIT 10 OFFSET 0 → id = 1~10

第2页：LIMIT 10 OFFSET 10 → id = 11~20

第一页：

| OFFSET | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    |
| ------ | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| id     | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   |

第二页：

| OFFSET | 10   | 11   | 12   | 13   | 14   | 15   | 16   | 17   | 18   | 19   |
| ------ | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| id     | 11   | 12   | 13   | 14   | 15   | 16   | 17   | 18   | 19   | 20   |



# 2. 并发插入

假设你在查完第一页后，在数据库中插入了 3条新的记录，它们的 id 是 100, 101, 102，并且这些记录会被排序排在前10条中间的位置。

比如你设置的是按 `name` 或其他字段排序，或者你用了时间戳字段排序，这三条记录插到了前面部分。

第一页已经把8、9、10 查出来并导出了，但第二页又重复查出来导出，造成**数据重复**（id = 7、8、9）

第一页：

| OFFSET | 0    | 1    | 2    | 3    | 4    | 5    | 6       | 7       | 8       | 9    |
| ------ | ---- | ---- | ---- | ---- | ---- | ---- | ------- | ------- | ------- | ---- |
| id     | 1    | 2    | 3    | 4    | 5    | 6    | **101** | **102** | **103** | 7    |

第二页：

| OFFSET | 10   | 11   | 12   | 13   | 14   | 15   | 16   | 17   | 18   | 19   |
| ------ | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| id     | 8    | 9    | 10   | 11   | 12   | 13   | 14   | 15   | 16   | 17   |

# 3. 并发删除

假设你在查询完第1页之后，在导出第2页之前，有人删除了第1页中的3条记录（id = 7、8、9）。

导致原本应该在第二页的记录（id=11、12、13）上移到了第一页，第二页查询的时候从id=14开始查，造成**数据丢失**（id=11、12、13）

第一页：

| OFFSET | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7      | 8      | 9      |
| ------ | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ------ | ------ | ------ |
| id     | 1    | 2    | 3    | 4    | 5    | 6    | 10   | **11** | **12** | **13** |

第二页：

| OFFSET | 10   | 11   | 12   | 13   | 14   | 15   | 16   | 17   | 18   | 19   |
| ------ | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| id     | 14   | 15   | 16   | 17   | 18   | 19   | 20   | 121  | 121  | 123  |

# 4. 问题解决

为了解决这个问题，可以考虑以下几种方案：

## 4.1. 方法一：使用游标分页（Cursor-based Pagination）

用上一页最后一条记录的 `id` 作为下一页的起点：

```sql
-- 第一页
SELECT * FROM inventory ORDER BY id LIMIT 10;

-- 第二页（假设最后一条是 id = 10）
SELECT * FROM inventory WHERE id > 10 ORDER BY id LIMIT 10;
```

这样无论有没有新增/删除记录，都不会影响下一页的准确性。

------

## 4.2. 方法二：快照导出（适用于大数据导出场景）

在导出前创建一个临时快照表或视图，把要导出的数据先复制一份：

```sql
CREATE TEMPORARY TABLE inventory_snapshot AS SELECT * FROM inventory;
```

然后对这个快照表进行分页查询，避免实时变化干扰。

## 4.3. 中间表 + 切面控制流程，构建了一个数据快照机制

不直接对库存表做分页查询，而是先将本次要导出的所有库存 ID 存入一个临时表中（export_record），后续导出都基于这个临时表的 ID 列表来进行。

```java
CREATE TABLE export_record (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    source_table_id BIGINT,        -- 来源表ID（如 inventory.id）
    batch_id VARCHAR(64),          -- 批次ID（异步任务ID）
    create_time DATETIME           -- 创建时间
);
@Service
public class InventoryExportService {

    @Autowired
    private InventoryMapper inventoryMapper;

    @Autowired
    private ExportRecordMapper exportRecordMapper;

    public void startExport() {
        String batchId = UUID.randomUUID().toString(); // 生成唯一批次ID

        // 获取所有需要导出的库存ID列表
        List<Long> allInventoryIds = getAllInventoryIds();

        // 将这些ID记录到export_record表中
        saveBatchIdsToExportRecord(batchId, allInventoryIds);

        // 异步执行实际导出逻辑
        asyncExportInventoryByBatchId(batchId);
    }

    private List<Long> getAllInventoryIds() {
        return inventoryMapper.selectList(new QueryWrapper<>())
        .stream()
        .map(Inventory::getId)
        .collect(Collectors.toList());
    }

    private void saveBatchIdsToExportRecord(String batchId, List<Long> ids) {
        List<ExportRecord> records = ids.stream()
        .map(id -> new ExportRecord(null, id, batchId, new Date()))
        .collect(Collectors.toList());
        exportRecordMapper.insertBatch(records); // 假设有一个批量插入的方法
    }

    @Async // 使用@Async注解实现异步调用
    public void asyncExportInventoryByBatchId(String batchId) {
        Page<ExportRecord> page = new Page<>(1, 1000); // 每次处理1000条记录
        IPage<ExportRecord> exportRecords;
        do {
            exportRecords = exportRecordMapper.selectPage(page, new QueryWrapper<ExportRecord>()
                                                          .eq("batch_id", batchId));

            if (exportRecords.getRecords().isEmpty()) break;

            // 根据export_records中的source_table_id查询inventory表
            List<Long> ids = exportRecords.getRecords().stream()
            .map(ExportRecord::getSourceTableId)
            .collect(Collectors.toList());
            List<Inventory> inventories = inventoryMapper.selectBatchIds(ids);

            // 执行导出逻辑（例如写入Excel文件）

            page.setCurrent(page.getCurrent() + 1); // 更新当前页码
        } while (true);

        // 清理中间表数据
        exportRecordMapper.delete(new QueryWrapper<ExportRecord>().eq("batch_id", batchId));
    }
}
```

### 1. 实现步骤

#### Step1：切面拦截导出请求

- 使用 Spring AOP 拦截带有 `@ExcelExport` 注解的方法
- 在方法执行前，获取到本次导出的库存 ID 列表（如从参数、上下文中提取）

```java
List<Long> inventoryIds = getInventoryIdsFromRequest();
```

#### Step2：生成批次标识 batchId，并写入通用导出表

- 将所有库存 ID 和 batchId 一起写入 `export_record` 表中

```java
INSERT INTO export_record (source_table_id, batch_id) VALUES
(1001, 'task_123'),
(1002, 'task_123'),
...
```

作用：保存本次导出所涉及的所有库存 ID。

#### Step3：导出逻辑改为根据 batchId 查询

- 导出代码不再直接查库存表，而是查 `export_record` 表中的 source_table_id 列表，再关联库存表导出

```sql
SELECT i.* FROM inventory i
JOIN export_record e ON i.id = e.source_table_id
WHERE e.batch_id = 'task_123'
ORDER BY i.id
LIMIT 10 OFFSET 0;
```

这样无论库存表如何变化，只要 `export_record` 表里的 ID 不变，导出的数据集就不会变。

#### Step4：导出完成后清理中间数据

- 在切面最后，删除该 batchId 对应的 `export_record` 数据，避免数据堆积

```sql
DELETE FROM export_record WHERE batch_id = 'task_123';
```

### 2. **核心原理**

核心原理：**固定数据集 + 独立于源表**

| **传统方式**                                          | **你的方案**                           |
| ----------------------------------------------------- | -------------------------------------- |
| 直接对库存表分页查询 → 数据会变                       | 先把 ID 写进中间表 → 数据不变          |
| 分页基于偏移量（OFFSET）→ 容易因插入/删除导致偏移错乱 | 分页基于固定 ID 列表 → 偏移始终准确    |
| 多次查询之间数据可能变动 → 导致重复或丢失             | 所有导出都基于同一份快照 ID → 数据一致 |

**解决了哪些问题？**

| **问题**             | **是否解决** | **原理说明**                               |
| -------------------- | ------------ | ------------------------------------------ |
| 数据插入导致分页偏移 | ✅ 解决       | 导出基于中间表 ID 列表，不受库存表插入影响 |
| 数据删除导致分页偏移 | ✅ 解决       | 同上，ID 快照已固定                        |
| 分页重复或丢失       | ✅ 解决       | 分页只针对固定的 ID 列表，不会跳过或重复   |
| 支持并发导出         | ✅ 支持       | 每个任务都有独立的 batchId，互不影响       |
| 可复用性             | ✅ 支持       | 中间表结构通用，适用于多种导出场景         |

------

### 3. 优化建议

1. **性能优化：批量插入中间表**

- 如果一次导出几万条记录，插入 `export_record` 表可能会慢。
- 建议使用 MyBatis 的 `saveBatch()` 或数据库的批量插入语句。

2. **清理机制保障**

- 确保每次导出完成后都能删除中间数据（即使异常也要删除）
- 可以加定时任务清理超时未删除的 `batch_id` 记录

3. **可扩展性增强**

- 可以考虑将 `export_record` 替换为 Redis 缓存，提升性能（短期缓存即可）
- 或者结合两者，热数据走 Redis，冷数据落库
