# 🪟 MySQL 窗口函数（Window Functions）笔记

---

# 一、窗口函数是什么？

## 1. 定义

👉 窗口函数：  
在**不减少行数的情况下**，对“当前行相关的一组数据”进行计算

---

## 2. 与 GROUP BY 的区别

| 特点 | GROUP BY | 窗口函数 |
|------|--------|----------|
| 行数 | 会减少 | 不变 |
| 结果 | 每组一行 | 每行都有结果 |
| 使用场景 | 汇总 | 分析 |

---

## 3. 核心理解

👉 每一行都有一个“窗口（window）”  
👉 窗口 = 与当前行相关的一组数据

---

# 二、窗口函数核心语法（必须掌握）

## ⭐ 标准语法


window_function(expression) OVER (
[PARTITION BY ...]
[ORDER BY ...]
[frame_clause]
)


---

## 1. function（函数部分）

### 分类：

#### （1）聚合函数
- SUM
- AVG
- COUNT
- MAX
- MIN

#### （2）排名函数
- ROW_NUMBER
- RANK
- DENSE_RANK

#### （3）分析函数
- LAG
- LEAD
- FIRST_VALUE
- LAST_VALUE

---

## 2. OVER

👉 标志这是窗口函数

---

## 3. PARTITION BY（分区）

### 作用：
👉 将数据划分为多个窗口（类似分组）

### 特点：
- 不减少行数
- 每个分区独立计算
- 会“重置计算结果”

---

## 4. ORDER BY（排序）

### 作用：
👉 定义窗口内数据顺序

### 影响：
- 排名函数必须依赖
- 累计计算依赖
- LAG/LEAD依赖

---

## 5. frame_clause（窗口范围）⭐️重点

👉 决定：当前行“能看到哪些行”

---

# 三、窗口范围（Frame）详解（重点）

## 1. 基本语法结构


ROWS BETWEEN 起点 AND 终点
RANGE BETWEEN 起点 AND 终点


---

## 2. Frame 起点/终点类型（必须掌握）

### （1）UNBOUNDED PRECEDING
👉 从第一行开始（向前无限）

### （2）UNBOUNDED FOLLOWING
👉 到最后一行（向后无限）

### （3）CURRENT ROW
👉 当前行

### （4）n PRECEDING
👉 当前行往前 n 行

### （5）n FOLLOWING
👉 当前行往后 n 行

---

# 四、Frame 常见组合（高频重点）

## 1️⃣ 累计窗口（默认）


UNBOUNDED PRECEDING → CURRENT ROW


👉 含义：
- 从第一行累计到当前行

👉 用途：
- 累计求和
- 累计平均

---

## 2️⃣ 向前滑动窗口


n PRECEDING → CURRENT ROW


👉 含义：
- 当前行 + 前 n 行

👉 用途：
- 移动平均
- 短期趋势分析

---

## 3️⃣ 对称滑动窗口


n PRECEDING → n FOLLOWING


👉 含义：
- 当前行前后各 n 行

👉 用途：
- 平滑数据

---

## 4️⃣ 向后窗口


CURRENT ROW → n FOLLOWING


👉 含义：
- 当前行 + 后 n 行

---

## 5️⃣ 全窗口


UNBOUNDED PRECEDING → UNBOUNDED FOLLOWING


👉 含义：
- 所有数据

👉 用途：
- 全局统计

---

# 五、ROWS vs RANGE（核心区别）

| 类型 | 控制方式 | 特点 |
|------|----------|------|
| ROWS | 行数 | 精确控制 |
| RANGE | 数值范围 | 会包含相同值 |

---

# 六、默认行为（必须理解）

## 1. 没有 PARTITION BY
👉 整表作为一个窗口

---

## 2. 有 ORDER BY 但没写 frame

默认：


RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW


👉 自动变成“累计”

---

## 3. 没有 ORDER BY
👉 无顺序概念 → 全部参与计算

---

# 七、执行顺序（容易考）


FROM → WHERE → GROUP BY → HAVING → 窗口函数 → ORDER BY


---

# 八、窗口函数分类（核心）

## 1️⃣ 聚合窗口函数
- 不减少行
- 每行都有聚合结果

---

## 2️⃣ 排名函数

| 函数 | 特点 |
|------|------|
| ROW_NUMBER | 唯一编号 |
| RANK | 会跳号 |
| DENSE_RANK | 不跳号 |

---

## 3️⃣ 分析函数

### LAG
👉 取前面的值

### LEAD
👉 取后面的值

### FIRST_VALUE
👉 窗口第一行

### LAST_VALUE
👉 窗口最后一行（受 frame 影响）

---

# 九、移动计算（核心应用）

## 移动平均
👉 滑动窗口平均

## 累计计算
👉 从起点累加到当前

---

# 十、排名与分布

### PERCENT_RANK
👉 排名占比

### CUME_DIST
👉 累计分布

---

# 十一、同比 / 环比 / 复合增长率

### 环比（MoM/QoQ）
👉 当前周期 vs 上一个周期
#### 公式
- 环比增长率 = （本期 - 上期）/ 上期

### 同比（YoY）
👉 当前周期 vs 去年同一周期
#### 公式
- 同比增长率 = （本期 - 去年同期）/去年同期

### 复合增长率（CAGR）
👉 一段时间内的“平均增长速度”
#### 公式
- CAGR = （第N期的数据 / 第1期的基准数据）的商，再开（N-1）次方，再减1

---

# 十二、WINDOW 子句（进阶）

## 作用
👉 复用窗口定义

## 语法
- WINDOW w AS (
-   PARTITION BY category
-   ORDER BY date
- )

## 示例
- SELECT
-    category
-    SUM(sales) OVER w AS total_sales,
-    AVG(sales) OVER w AS avg_sales
- FROM orders
- WINDOW w AS (
-    PARTITION BY category
-    ORDER BY date
- )

## 本质
👉 给窗口命名

---

# 十三、关键总结（必须背）

## ⭐ 核心结构


函数 + OVER(分区 + 排序 + 窗口范围)


---

## ⭐ 三大控制

1. PARTITION BY → 分组  
2. ORDER BY → 顺序  
3. FRAME → 范围（灵魂）

---

## ⭐ Frame 记忆口诀

- UNBOUNDED PRECEDING（开头）
- UNBOUNDED FOLLOWING（结尾）
- CURRENT ROW（当前）
- n PRECEDING（向前）
- n FOLLOWING（向后）

---

## ⭐ 本质理解

👉 窗口函数 =  
让每一行在“指定范围的数据窗口中”进行计算

---

# 🚀 一句话总结

👉 窗口函数真正核心：

> 控制“每一行能看到的数据范围”，然后进行计算
