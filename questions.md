# SQL 面试高频 20 题 — 软件测试岗

> 基于缺陷管理系统数据，贴近测试日常工作。
> 先跑 `mysql -u root -p < setup.sql` 建库，再逐题练习。

---

## 基础查询（必考）

### Q1. 查询所有致命级别（severity='致命'）的缺陷，按创建时间降序

```sql
SELECT id, title, status, created_date
FROM bugs
WHERE severity = '致命'
ORDER BY created_date DESC;
```

**考点**：WHERE + ORDER BY。面试官要看你是否知道 DESC。

---

### Q2. 查询状态为「已关闭」或「已修复」的缺陷，用 IN

```sql
SELECT title, severity, status
FROM bugs
WHERE status IN ('已关闭', '已修复');
```

**考点**：IN vs OR。IN 更简洁，大数据量时 IN 优化更好。

---

### Q3. 查询标题含「登录」关键词的缺陷（模糊查询）

```sql
SELECT id, title, status
FROM bugs
WHERE title LIKE '%登录%';
```

**考点**：LIKE + 通配符。`%` 匹配任意字符，`_` 匹配单字符。面试追问：「%登录% 能走索引吗？」→ 前缀 `%` 不走。

---

### Q4. 查询每个严重等级的缺陷数量（GROUP BY）

```sql
SELECT severity, COUNT(*) AS bug_count
FROM bugs
GROUP BY severity
ORDER BY bug_count DESC;
```

**考点**：GROUP BY + 聚合函数。面试追问：「WHERE 和 HAVING 区别？」→ WHERE 在分组前过滤行，HAVING 在分组后过滤组。

---

### Q5. 在上题基础上，只显示数量 ≥3 的等级（HAVING）

```sql
SELECT severity, COUNT(*) AS bug_count
FROM bugs
GROUP BY severity
HAVING COUNT(*) >= 3;
```

**考点**：HAVING 用法。聚合条件写在 HAVING 而非 WHERE。

---

## 多表查询（高频）

### Q6. 查询每个缺陷的标题、严重等级，以及所属项目名称（INNER JOIN）

```sql
SELECT b.id, b.title, b.severity, p.name AS project_name
FROM bugs b
INNER JOIN projects p ON b.project_id = p.id;
```

**考点**：INNER JOIN 语法。面试追问：「还有什么 JOIN？」→ LEFT/RIGHT/CROSS/SELF。

---

### Q7. 列出所有项目及其缺陷数量，包括没有缺陷的项目（LEFT JOIN）

```sql
SELECT p.name, COUNT(b.id) AS bug_count
FROM projects p
LEFT JOIN bugs b ON p.id = b.project_id
GROUP BY p.id, p.name;
```

**考点**：LEFT JOIN + GROUP BY。LEFT JOIN 保留左表所有行。

---

### Q8. 查询缺陷标题 + 发现人姓名 + 处理人姓名（自连接 x2）

```sql
SELECT b.title,
       t1.name AS 发现人,
       t2.name AS 处理人
FROM bugs b
JOIN testers t1 ON b.tester_id = t1.id
LEFT JOIN testers t2 ON b.assignee_id = t2.id;
```

**考点**：同一张表 JOIN 两次 — 用不同别名区分。处理人可能为 NULL（如「新建」状态），用 LEFT JOIN。

---

### Q9. 查询每个测试人员发现的缺陷数（含未发现缺陷的人）

```sql
SELECT t.name, COUNT(b.id) AS found_bugs
FROM testers t
LEFT JOIN bugs b ON t.id = b.tester_id
GROUP BY t.id, t.name
ORDER BY found_bugs DESC;
```

**考点**：LEFT JOIN 保留零值 + COUNT 不统计 NULL 的特性。

---

## 子查询

### Q10. 查询严重等级高于「一般」的所有缺陷（用子查询，不用 IN）

```sql
-- 思路：子查询不等于「一般」，但多等级比较用 IN 更直观。
-- 本题示范子查询结构：

SELECT title, severity
FROM bugs
WHERE severity IN ('致命', '严重');

-- 或等价子查询写法（面试展示用）：
SELECT title, severity
FROM bugs
WHERE id IN (
    SELECT id FROM bugs WHERE severity IN ('致命', '严重')
);
```

**考点**：子查询 = 查询中的查询。面试追问：「子查询 vs JOIN 哪个快？」→ 看场景，现代优化器通常处理成一样的执行计划。

---

### Q11. 查询发现的缺陷数 > 平均缺陷数的项目

```sql
SELECT p.name, COUNT(b.id) AS cnt
FROM projects p
JOIN bugs b ON p.id = b.project_id
GROUP BY p.id, p.name
HAVING cnt > (
    SELECT AVG(cnt) FROM (
        SELECT COUNT(*) AS cnt FROM bugs GROUP BY project_id
    ) AS sub
);
```

**考点**：子查询嵌套 + HAVING 比较聚合值。这是面试高分段题目。

---

## 聚合 + 排序 + 分页

### Q12. 查询每个项目的缺陷关闭率（已关闭数 / 总数）

```sql
SELECT p.name,
       COUNT(b.id) AS total,
       SUM(CASE WHEN b.status = '已关闭' THEN 1 ELSE 0 END) AS closed,
       ROUND(SUM(CASE WHEN b.status = '已关闭' THEN 1 ELSE 0 END) / COUNT(b.id) * 100, 1) AS close_rate
FROM projects p
LEFT JOIN bugs b ON p.id = b.project_id
GROUP BY p.id, p.name;
```

**考点**：CASE WHEN 在聚合中的用法。面试常问：「MySQL 怎么行转列？」→ 就是 CASE WHEN + GROUP BY。

---

### Q13. 查询 2026 年 3 月创建的缺陷，按日期分组统计

```sql
SELECT created_date, COUNT(*) AS cnt
FROM bugs
WHERE created_date BETWEEN '2026-03-01' AND '2026-03-31'
GROUP BY created_date
ORDER BY created_date;
```

**考点**：日期范围 + 分组。

---

### Q14. 查缺陷总数第 3-5 多的项目（分页 LIMIT OFFSET）

```sql
SELECT p.name, COUNT(b.id) AS cnt
FROM projects p
LEFT JOIN bugs b ON p.id = b.project_id
GROUP BY p.id, p.name
ORDER BY cnt DESC
LIMIT 3 OFFSET 2;   -- 跳过前2条，取3条
```

**考点**：LIMIT + OFFSET 分页。面试追问：「OFFSET 大有什么问题？」→ 扫描跳过行，性能差 → 用游标分页或 WHERE id > last_id。

---

## 增删改 + 表结构（笔试题高频）

### Q15. 给 bugs 表新增一个字段 `fix_version` VARCHAR(20)

```sql
ALTER TABLE bugs ADD COLUMN fix_version VARCHAR(20);
```

---

### Q16. 更新缺陷 #5 的处理人为「王五」(id=3)，状态改为「已修复」

```sql
UPDATE bugs
SET assignee_id = 3, status = '已修复'
WHERE id = 5;
```

**考点**：UPDATE 多字段。**面试追问**：「如果忘了 WHERE 会怎样？」→ 全表更新，灾难。测试环境写 UPDATE 前先 SELECT 确认范围。

---

### Q17. 删除所有「轻微」级别且状态为「新建」的缺陷

```sql
DELETE FROM bugs
WHERE severity = '轻微' AND status = '新建';
```

**考点**：DELETE + 多条件 AND。追问：「DELETE vs TRUNCATE vs DROP 区别？」
- DELETE：删行，可回滚，可加 WHERE，触发器生效
- TRUNCATE：清空表，不可回滚，重置自增，快
- DROP：删除整个表（结构+数据）

---

### Q18. 插入一条新缺陷

```sql
INSERT INTO bugs (title, severity, status, project_id, tester_id, created_date)
VALUES ('注册页验证码不刷新', '严重', '新建', 1, 2, CURDATE());
```

**考点**：INSERT 语法 + CURDATE() 函数。

---

## DISTINCT + UNION（进阶）

### Q19. 查询所有出现在 bugs 表中的 tester_id 和 assignee_id 的并集（去重）

```sql
SELECT tester_id AS person_id FROM bugs
UNION
SELECT assignee_id FROM bugs WHERE assignee_id IS NOT NULL;
```

**考点**：UNION（去重并集）vs UNION ALL（不去重）。去重有排序开销，确定不重复时用 UNION ALL。

---

### Q20. 综合报表：每个项目的测试概况

```sql
SELECT
    p.name AS 项目名称,
    COUNT(DISTINCT b.id) AS 缺陷总数,
    COUNT(DISTINCT CASE WHEN b.status = '已关闭' THEN b.id END) AS 已关闭,
    COUNT(DISTINCT tc.id) AS 用例总数,
    COUNT(DISTINCT CASE WHEN tc.result = '通过' THEN tc.id END) AS 通过用例,
    COUNT(DISTINCT CASE WHEN tc.result = '失败' THEN tc.id END) AS 失败用例
FROM projects p
LEFT JOIN bugs b ON p.id = b.project_id
LEFT JOIN test_cases tc ON p.id = tc.project_id
GROUP BY p.id, p.name;
```

**考点**：多表 LEFT JOIN + COUNT DISTINCT + CASE WHEN 组合。这题答出来，面试官会觉得你 SQL 过关。

---

## 面试应答话术

| 问题 | 回答 |
|------|------|
| "你 SQL 什么水平？" | 能独立写多表 JOIN + 子查询，工作中用 MySQL 做过缺陷统计报表 |
| "用过哪些聚合函数？" | COUNT/SUM/AVG/MAX/MIN，配合 GROUP BY + HAVING 做分组统计 |
| "JOIN 有几种？" | INNER/LEFT/RIGHT/CROSS。LEFT JOIN 最常用，比如查所有项目的缺陷数（含无缺陷的） |
| "优化过慢查询吗？" | 用 EXPLAIN 看执行计划，主要加索引、避免 SELECT *、LIMIT 分页优化 |
