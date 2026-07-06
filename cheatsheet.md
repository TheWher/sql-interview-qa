# SQL 面试速查卡 — 5 分钟过一遍

## 执行顺序（面试官爱考）

```text
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

手写 SQL 顺序：`SELECT ... FROM ... JOIN ... WHERE ... GROUP BY ... HAVING ... ORDER BY ... LIMIT`

---

## 聚合函数

| 函数 | 用途 |
|------|------|
| `COUNT(*)` | 行数（含 NULL） |
| `COUNT(col)` | 非 NULL 行数 |
| `SUM(col)` | 求和 |
| `AVG(col)` | 平均 |
| `MAX(col)` / `MIN(col)` | 最大/最小 |

---

## WHERE vs HAVING

| | WHERE | HAVING |
|------|-------|--------|
| 作用对象 | 行 | 组 |
| 执行时机 | 分组前 | 分组后 |
| 聚合函数 | ❌ 不能 | ✅ 能 |

---

## JOIN 速查

```
INNER JOIN  → 两表都有
LEFT JOIN   → 左表全保留
RIGHT JOIN  → 右表全保留
CROSS JOIN  → 笛卡尔积（少用）
```

---

## 常用函数

| 函数 | 示例 |
|------|------|
| `LIKE` | `WHERE name LIKE '%测试%'` |
| `IN` | `WHERE status IN ('新建','已分配')` |
| `BETWEEN` | `WHERE date BETWEEN '2026-01' AND '2026-06'` |
| `DISTINCT` | `SELECT DISTINCT severity FROM bugs` |
| `IFNULL` | `IFNULL(closed_date, '未关闭')` |
| `COALESCE` | `COALESCE(col1, col2, '默认')` |
| `LIMIT` | `LIMIT 10 OFFSET 20` |
| `CASE` | `CASE WHEN status='已关闭' THEN 1 ELSE 0 END` |

---

## 常见陷阱

| 陷阱 | 正确做法 |
|------|---------|
| NULL 用 `=` 比较 | `IS NULL` / `IS NOT NULL` |
| DELETE 忘 WHERE | 先 SELECT 确认范围 |
| GROUP BY 后 SELECT 非分组列 | 只用分组列 + 聚合函数 |
| COUNT 想排重 | `COUNT(DISTINCT col)` |
| UNION 自动去重慢 | 确定不重复用 `UNION ALL` |
| LIKE `%xxx%` 前缀通配符 | 不走索引，数据量大用全文索引 |

---

## DELETE / TRUNCATE / DROP

| | DELETE | TRUNCATE | DROP |
|------|--------|----------|------|
| 删除 | 行 | 全表数据 | 表结构+数据 |
| 回滚 | ✅ | ❌ | ❌ |
| WHERE | ✅ | ❌ | ❌ |
| 自增重置 | ❌ | ✅ | — |

---

## 面试最后一问（加分项）

> "为什么 COUNT(*) 和 COUNT(col) 结果可能不同？"

答：`COUNT(*)` 统计所有行（含全 NULL 行）；`COUNT(col)` 只统计该列非 NULL 的行。如果 col 有 NULL 值，`COUNT(col)` 会比 `COUNT(*)` 少。
