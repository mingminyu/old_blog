# 排序

<show-structure depth="2"/>

## 1. 随机排序

在 Presto 中可以通过 `ORDER BY RAND()` 来对数据进行随机排序，结合 `LIMIT n` 语句从而达到随机采样 `n` 条数据的效果，以下是对表中数据随机筛选 1000 条 SQL 代码示例：

```SQL
SELECT * FROM <YOUR_TABLE>
ORDER BY RAND()
LIMIT 1000
```

> 如果你的主查询是 `SELECT DISTINCT ...` 语句的话，那么不可以和 `ORDER BY RAND()` 直接连用，需要先将主查询中 `SELECT DISTINCT` 子查询的形式。
{style="warning"}


