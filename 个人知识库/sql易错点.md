# 🚨 SQL 五大致命坑（数据分析必踩雷区）

> 适用于：数据分析 / 电商分析 / 面试  
> 目标：避免“结果看起来对，其实是错的”

---

# ❌ 坑1：CROSS JOIN 造成数据膨胀

## 🔥 问题

```sql
FROM A
CROSS JOIN B
```

👉 会产生：

```
行数 = A × B
```

### 💥 后果

- 数据被“复制”
- 聚合结果被稀释或放大
- 出现莫名其妙的比例错误

### 🚫 典型错误

```sql
SELECT *
FROM Queries q
CROSS JOIN (
    SELECT COUNT(*) AS c FROM Queries
) t
```

👉 每一行都配上所有 c → 错！

### ✅ 正确做法

👉 必须有“关联条件”

```sql
JOIN ... ON A.xxx = B.xxx
```

---

# ❌ 坑2：聚合后丢失维度（最常见）

## 🔥 问题

```sql
SELECT COUNT(*) 
FROM Queries 
GROUP BY query_name
```

👉 结果只有：

```
c
```

👉 没有 query_name！

### 💥 后果

- 无法 JOIN 回原表
- 数据错位（A 用了 B 的数据）

### 🚫 错误示例

```sql
SELECT COUNT(*) AS c
FROM Queries
GROUP BY query_name
```

👉 ❌ 丢了 query_name

### ✅ 正确写法

```sql
SELECT query_name, COUNT(*) AS c
FROM Queries
GROUP BY query_name
```

### 🎯 口诀

分组字段必须带出来！

---

# ❌ 坑3：LEFT JOIN 被 WHERE “干掉”

## 🔥 问题

```sql
SELECT *
FROM A
LEFT JOIN B ON A.id = B.id
WHERE B.id IS NOT NULL
```

### 💥 实际效果

👉 等价于：

```sql
INNER JOIN
```

👉 LEFT JOIN 白写了

### ✅ 正确写法

```sql
LEFT JOIN B 
ON A.id = B.id
AND B.xxx = ...
```

### 🎯 口诀

LEFT JOIN 的过滤条件写 ON，不写 WHERE

---

# ❌ 坑4：WHERE 和 HAVING 用错

## 🔥 区别

|        | WHERE | HAVING |
|--------|--------|--------|
| 执行时机 | 分组前 | 分组后 |
| 聚合函数 | ❌ | ✅ |

### 🚫 错误写法

```sql
SELECT query_name
FROM Queries
WHERE COUNT(*) > 1
```

👉 ❌ 报错

### ✅ 正确写法

```sql
SELECT query_name
FROM Queries
GROUP BY query_name
HAVING COUNT(*) > 1
```

### 🎯 口诀

看到 COUNT / SUM → 用 HAVING

---

# ❌ 坑5：把“分组问题”写成“子查询 + JOIN”

## 🔥 问题

👉 明明可以 GROUP BY，一定要写子查询

### 🚫 错误写法

```sql
SELECT q.query_name, ff.c
FROM Queries q
JOIN (
    SELECT query_name, COUNT(*) AS c
    FROM Queries
    GROUP BY query_name
) ff
ON q.query_name = ff.query_name
```

### 💥 问题

- 复杂
- 易错
- 容易重复计算

### ✅ 正确写法（最优）

```sql
SELECT 
    query_name,
    COUNT(*) AS c
FROM Queries
GROUP BY query_name
```

---

### 🚀 高级写法（推荐）

```sql
SUM(condition)
```

例如：

```sql
SUM(rating < 3)
```

👉 等价于：

差评数量

### 🎯 口诀

能 GROUP BY 就别 JOIN  
能 SUM(condition) 就别子查询  

---

# 🧠 总结（核心认知）

✅ SQL 本质是三件事：

1. 行 → WHERE  
2. 分组 → GROUP BY  
3. 结果 → HAVING / SELECT  

---

# 🚀 终极口诀（强烈建议记住）

JOIN 是“对齐数据”  
GROUP BY 是“做统计”  
子查询是“不得已”  

---

# 🎯 适用于你当前阶段的建议

你现在已经进入：

会写 SQL ✅  
但容易选错解法 ⚠️  

👉 这是进阶的必经阶段 👍  

---

## 📌 如果你继续练习

优先掌握这几类：

- 去重（DISTINCT / 自连接）
- TopN（ROW_NUMBER）
- 首次 / 最新（MIN / MAX）
- 时间对比（LAG / 自连接）
- 占比（SUM(condition)）
