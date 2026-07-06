# SQL 面试题库 — 软件测试岗

> 20 道高频题 + 建表脚本 + 5 分钟速查卡
> 数据领域：缺陷管理系统（贴近测试日常工作）

## 快速开始

```bash
# 1. 建库建表
mysql -u root -p < setup.sql

# 2. 对照 questions.md 逐题练习
```

## 内容

| 文件 | 说明 |
|------|------|
| `setup.sql` | 4 张表（缺陷/用例/项目/测试人员）+ 31 条测试数据 |
| `questions.md` | 20 题：从 WHERE 到多表 JOIN + 子查询，含答案 + 考点 + 面试追问 |
| `cheatsheet.md` | 面试前 5 分钟速查：执行顺序/聚合函数/JOIN 对比/DELETE 三兄弟 |

## 覆盖考点

`WHERE` `LIKE` `GROUP BY` `HAVING` `INNER JOIN` `LEFT JOIN` `子查询` `CASE WHEN` `COUNT DISTINCT` `UNION` `LIMIT OFFSET` `ALTER TABLE` `UPDATE` `DELETE` `INSERT`

## 为什么不是"学生表"

题库数据模拟真实**缺陷管理系统**：bugs / test_cases / projects / testers，面试时说"我用 SQL 做过缺陷统计报表"比"我查过学生成绩表"靠谱得多。
