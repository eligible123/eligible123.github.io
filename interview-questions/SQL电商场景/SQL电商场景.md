---
title: SQL电商场景面试题
date: 2026-06-07
type: page
---

# SQL电商场景面试题

> 共 34 道面试题

## 1. 编写 SQL，按创建时间升序查询所有客户

🟢 **简单** | 标签：SQL, SQL基础, SQL电商场景

```sql
SELECT * FROM customers ORDER BY created_at ASC;
```

**核心要点说明：**
1. **排序逻辑**：使用 `ORDER BY created_at ASC` 对 `created_at` 字段进行升序排列（ASC为升序，可省略；DESC为降序）。这确保最早注册的客户排在最前面。

2. **电商场景应用建议**：
   - 实际业务中建议配合 `WHERE` 条件进行时间范围筛选（如最近30天注册客户）。
   - 若数据量较大，可在 `created_at` 字段上建立索引以优化查询性能。
   - 需确认时间字段是否为有效时间类型（如TIMESTAMP/DATETIME），避免字符串类型导致排序错误。

**回答示例：**
“我会编写一条基础查询语句，使用 `ORDER BY created_at ASC` 按创建时间升序排列客户。在电商场景中，这通常用于分析用户注册趋势，例如结合 `WHERE created_at > '2023-01-01'` 获取今年新增客户。需要注意确保时间字段类型正确，并建议对大表添加索引优化查询效率。”

---

## 2. 编写 SQL，查询价格最高的商品信息

🟢 **简单** | 标签：SQL, SQL基础, SQL电商场景

该问题核心在于找到价格等于最高价格的商品。常用两种思路，推荐第一种以覆盖所有并列情况。

**核心思路**：先获取最高价格，再查询价格等于该值的商品。

**标准写法（使用子查询）**：
```sql
SELECT * FROM product
WHERE price = (SELECT MAX(price) FROM product);
```
*说明*：子查询 `(SELECT MAX(price) FROM product)` 获取最高价格，主查询筛选出所有价格等于该值的记录，能正确返回所有最高价商品。

**简洁写法（使用排序与限制）**：
```sql
SELECT * FROM product ORDER BY price DESC LIMIT 1;
```
*注意*：此方法仅返回一条记录。若最高价有多个商品，则随机返回其一。仅适用于单行结果的场景，通用性较弱。

**面试要点**：
1.  **子查询优先**：标准写法逻辑清晰，且能处理多个商品价格相同并列最高的情况。
2.  **注意边界**：面试时应主动说明两种写法的区别，尤其是第二个方法在并列最高价时的行为，这能体现思考的严谨性。
3.  **性能考量**：对于大数据表，两种写法均可利用索引。子查询方法会扫描表两次，但在现代数据库优化下通常表现良好。

---

## 3. 编写 SQL，查询创建时间在 2023 年 2 月份的所有客户

🟢 **简单** | 标签：SQL, SQL基础, SQL电商场景

```sql
SELECT *
FROM customers
WHERE create_time >= '2023-02-01'
  AND create_time < '2023-03-01';
```

**核心要点解析：**

1.  **条件表达式**：使用 `WHERE` 子句筛选 `create_time` 字段。
2.  **时间范围界定**：为精确查询整个二月份数据，采用 **“大于等于起始日”** 且 **“严格小于下月第一天”** 的半开区间。这比使用 `BETWEEN`（可能遗漏2月28日23:59:59后的数据）或按月函数匹配更严谨高效。
3.  **性能与兼容性**：此写法直接利用索引（若`create_time`已建索引），查询效率高。若数据库支持（如MySQL），可使用 `DATE_FORMAT(create_time, '%Y-%m') = '2023-02'` 简化，但需注意函数可能导致索引失效，应视情况选择。

---

## 4. 编写 SQL，查询总金额大于 100 的所有订单

🟢 **简单** | 标签：SQL, SQL基础, SQL电商场景

查询订单总金额大于100的SQL语句如下：

```sql
SELECT
    order_id,
    SUM(amount) AS total_amount
FROM
    order_items  -- 假设金额明细在订单商品表中
GROUP BY
    order_id
HAVING
    SUM(amount) > 100;
```

**核心要点解析：**
1.  **业务理解**：通常电商数据库中，单个订单（order）由多个订单商品项（order_items）组成。因此，计算“订单总金额”需要对商品明细进行汇总。
2.  **关键子句**：
    *   `SUM(amount)`：对每个订单的所有商品金额进行聚合求和。
    *   `GROUP BY order_id`：按订单ID进行分组，以便计算每个独立订单的总和。
    *   `HAVING`：对分组聚合后的结果（`SUM(amount)`）进行条件过滤，`WHERE`子句无法用于聚合函数。
3.  **数据完整性建议**：为确保计算准确，可添加 `WHERE amount IS NOT NULL` 预先过滤空值，或使用 `COALESCE(amount, 0)` 将空值视为0参与计算。
4.  **性能提示**：确保对`order_id`字段建有索引，以优化分组查询效率。

**进阶思考**：若需关联显示订单其他信息（如客户、日期），可将上述查询作为子查询与订单主表进行JOIN操作。

---

## 5. 编写 SQL，查询所有客户的姓名和创建时间，结果按照创建时间降序排序

🟢 **简单** | 标签：SQL, SQL基础, SQL电商场景

```sql
SELECT name, create_time
FROM customers
ORDER BY create_time DESC;
```

**要点说明：**

1. **核心语法**：使用 `SELECT` 指定查询字段（`name`, `create_time`），`FROM` 指定数据表（`customers`）。
2. **排序逻辑**：`ORDER BY create_time DESC` 按创建时间降序排列，最新创建的客户排在前面。
3. **实际场景**：在电商中，此查询常用于分析新客户趋势或优先处理近期注册的用户，例如在客服系统或会员管理中查看最新注册客户。
4. **优化提示**：若数据量较大，可为 `create_time` 字段添加索引以提升排序性能。

---

## 6. 编写 SQL，查询所有客户的姓名和电话

🟢 **简单** | 标签：SQL, SQL电商场景, SQL基础

SELECT name, phone FROM customers;

该SQL语句执行以下操作：
1. 指定从customers（客户表）中查询数据
2. 选择name（姓名）和phone（电话）两个字段
3. 因未添加WHERE条件，将返回所有客户记录

关键点说明：
- 若表中字段名为中文（如“姓名”“电话”），需使用反引号包裹：`姓名`, `电话`
- 实际应用中建议添加分页限制（如LIMIT 100）防止数据量过大
- 可通过EXPLAIN分析查询性能，确保索引优化

---

## 7. 编写 SQL，查询所有客户的姓名，并将其姓名转换为大写

🟢 **简单** | 标签：SQL, SQL基础, SQL电商场景

# SQL查询：客户姓名大写转换

## SQL代码

```sql
SELECT UPPER(customer_name) AS customer_name_upper
FROM customers;
```

## 代码说明

**核心函数：`UPPER()`**
- 该函数用于将字符串转换为大写形式
- 是SQL标准字符串函数，主流数据库（MySQL、PostgreSQL、Oracle、SQL Server）均支持

**语句解析：**
| 部分 | 作用 |
|------|------|
| `SELECT UPPER(customer_name)` | 对客户姓名字段应用大写转换 |
| `AS customer_name_upper` | 为结果列设置别名，便于识别 |
| `FROM customers` | 指定数据来源表 |

## 补充说明

**相关函数对比：**
- `UPPER()` - 转大写
- `LOWER()` - 转小写
- `INITCAP()` - 首字母大写（Oracle/PostgreSQL支持）

**实际应用场景：**
- 数据规范化：统一存储格式，便于后续模糊查询和数据比对
- 报表展示：满足特定业务需求的显示格式
- 数据导入：对接外部系统时的格式要求

**扩展写法（如需筛选条件）：**
```sql
SELECT UPPER(customer_name) AS customer_name_upper
FROM customers
WHERE status = 'active';
```

此题考查的是对字符串处理函数的掌握，属于SQL基础能力，实际开发中常用于数据清洗和格式统一。

---

## 8. 编写 SQL，查询所有订单的总金额和订单日期

🟢 **简单** | 标签：SQL, SQL基础, SQL电商场景

假设订单表 `orders` 包含字段：`order_id`（订单ID）、`order_date`（订单日期）、`total_amount`（订单总金额）。若需计算所有订单的总金额（所有订单累加）及对应日期，有两种常见场景：

**场景一：查询每笔订单的金额与日期**
```sql
SELECT order_date, total_amount 
FROM orders;
```
此查询直接返回每条订单的日期及其金额，适用于明细查看。

**场景二：计算每日所有订单的总额（聚合查询）**
```sql
SELECT order_date, SUM(total_amount) AS daily_total_amount
FROM orders
GROUP BY order_date
ORDER BY order_date;
```
此查询按日期分组，求和得到每日总金额，并按日期排序，是电商分析中常见的日销售汇总查询。

**核心要点**：
1. **明确需求**：区分是查询订单明细还是进行聚合汇总。
2. **聚合函数**：涉及汇总时需使用 `SUM()`、`COUNT()` 等。
3. **分组与排序**：使用 `GROUP BY` 实现分类聚合，`ORDER BY` 保证结果有序。

若表中存储的是商品明细（如 `order_items` 表含 `order_id`、`product_price`、`quantity`），则需先通过 `JOIN` 或子查询计算每单总金额，再与 `orders` 表关联获取日期。

---

## 9. 编写 SQL，查询最早的订单日期

🟢 **简单** | 标签：SQL, SQL基础, SQL电商场景

**答案：**

查询最早的订单日期，通常使用聚合函数 `MIN()` 来实现。核心SQL语句如下：

```sql
SELECT MIN(order_date) AS earliest_order_date FROM orders;
```

**核心要点解析：**

1.  **函数选择**：`MIN(order_date)` 函数直接从 `order_date` 列中找出最小值，即最早的日期。这是最直接、高效的写法。
2.  **数据来源**：`FROM orders` 指明数据从订单表中获取。需根据实际表名调整。
3.  **条件过滤（可选）**：若需查询特定条件（如某用户、某商品）的最早订单，可在 `WHERE` 子句中添加筛选条件，例如：`WHERE user_id = 1001`。
4.  **别名可读性**：`AS earliest_order_date` 为结果列设置了清晰的别名，提升结果的可读性。

**扩展说明：**
另一种思路是按日期排序并取第一条记录，如 `SELECT order_date FROM orders ORDER BY order_date ASC LIMIT 1;`。但在标准SQL中，使用 `MIN()` 函数在语义上更明确，在数据库内部优化上也通常更高效。

**注意：** 确保 `order_date` 列为日期或时间戳类型。若该列可能包含 `NULL` 值，`MIN()` 函数会自动忽略 `NULL`，只从非 `NULL` 值中计算最小值。

---

## 10. 编写 SQL，查询每个客户的订单总金额，并按总金额降序排序

🟢 **简单** | 标签：SQL, SQL进阶, SQL电商场景

## 解题思路与SQL实现

### 1. 假设表结构

```sql
-- 客户表 customers
customer_id (客户ID), customer_name (客户姓名)

-- 订单表 orders
order_id (订单ID), customer_id (客户ID), amount (订单金额)
```

### 2. SQL实现

```sql
SELECT 
    c.customer_id,
    c.customer_name,
    SUM(o.amount) AS total_amount
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name
ORDER BY total_amount DESC;
```

### 3. 核心要点说明

**分组聚合**：使用 `GROUP BY` 按客户分组，`SUM()` 计算每个客户的订单总金额。

**多表连接**：通过 `JOIN` 关联客户表和订单表，使用 `ON` 指定连接条件。

**排序**：`ORDER BY total_amount DESC` 实现降序排列，金额最高的客户排在前面。

### 4. 进阶补充

- 若需显示未下单客户，改用 `LEFT JOIN` 并处理 NULL：`COALESCE(SUM(o.amount), 0)`
- 若只需前N名，可添加 `LIMIT N`
- 注意：`GROUP BY` 中的字段应与 `SELECT` 中的非聚合字段保持一致，这是SQL规范要求

---

## 11. 编写 SQL，查询每个订单的总商品数量

🟢 **简单** | 标签：SQL, SQL基础, SQL电商场景

在电商场景中，查询每个订单的总商品数量是常见的SQL操作，通常基于订单详情表（例如 `order_details`），该表记录每个订单中的商品及其数量。核心思路是使用聚合函数 `SUM()` 对数量字段求和，并通过 `GROUP BY` 按订单分组。

以下是标准的SQL查询示例：

```sql
SELECT 
    order_id,
    SUM(quantity) AS total_items
FROM 
    order_details
GROUP BY 
    order_id;
```

**核心要点解析**：
- `SELECT order_id`：指定输出订单标识，用于区分不同订单。
- `SUM(quantity)`：聚合函数，计算每个订单内所有商品的数量总和，并通过 `AS total_items` 设置别名以提高可读性。
- `FROM order_details`：指定数据来源表，假设表包含 `order_id`（订单ID）和 `quantity`（商品数量）字段。
- `GROUP BY order_id`：关键子句，将数据按订单ID分组，确保每个订单独立计算总数量。

此查询适用于基础表结构。若需关联订单主表（如 `orders`）获取更多订单信息，可使用 `JOIN` 扩展，但题目仅要求总数量，故此查询已满足需求。在实际面试中，建议先确认表结构，再根据上下文调整查询。

---

## 12. 编写 SQL，查询订单总金额的平均值

🟢 **简单** | 标签：SQL, SQL基础, SQL电商场景

针对该问题，提供以下两种常见场景的SQL写法，核心在于准确计算“订单总金额”并使用聚合函数`AVG()`。

**场景一：假设订单表（orders）中已存储每个订单的总金额字段（如`total_amount`）**
```sql
SELECT AVG(total_amount) AS avg_order_total
FROM orders;
```
此情况最简单，直接对`total_amount`列使用`AVG()`函数即可得到所有订单总金额的平均值。

**场景二：订单总金额需从订单明细表（order_details）中计算得出**
此时需先计算每个订单的总金额，再求平均。
```sql
SELECT AVG(sub.total_order_amount) AS avg_order_total
FROM (
    SELECT order_id, SUM(quantity * price) AS total_order_amount
    FROM order_details
    GROUP BY order_id
) AS sub;
```
**核心要点说明**：
1. **子查询计算单个订单总金额**：通过`GROUP BY order_id`和`SUM(quantity * price)`，得出每个订单的总金额。
2. **外层聚合计算平均值**：外层查询对子查询结果中的`total_order_amount`列使用`AVG()`函数，计算所有订单总金额的平均值。
3. **命名与可读性**：使用`AS`为计算结果列起别名，增强SQL可读性。此结构清晰展示了先聚合明细、再二次聚合的逻辑，是电商场景中处理多级聚合问题的典型范例。

---

## 13. 编写 SQL，统计共有多少客户

🟢 **简单** | 标签：SQL, SQL基础, SQL电商场景

统计客户总数的 SQL 如下：

```sql
-- 方法一：直接统计客户表
SELECT COUNT(customer_id) AS total_customers
FROM customers;

-- 方法二：若需统计有过订单记录的客户
SELECT COUNT(DISTINCT customer_id) AS total_customers
FROM orders;
```

**核心要点说明**：
1. **统计维度**：若“客户”指已注册用户，通常直接统计 `customers` 表；若指有购买行为的用户，则需关联 `orders` 表。
2. **去重处理**：使用 `COUNT(DISTINCT customer_id)` 可确保每个客户仅计数一次，避免因同一客户多次下单而重复统计。
3. **字段选择**：优先选择主键或唯一标识字段（如 `customer_id`），避免因数据不规范导致计数偏差。
4. **场景适配**：实际业务中常需按时间、地区等维度进一步筛选，可结合 `WHERE` 子句添加条件。

---

## 14. 编写 SQL，查询在 2023 年 1 月份下的所有订单及其对应的商品详细信息

🟡 **中等** | 标签：SQL, SQL进阶, SQL电商场景

查询2023年1月份所有订单及其商品信息，需通过订单表与商品表关联实现。以下是两种常用写法：

**方法一：直接时间范围匹配**  
```sql
SELECT 
    o.order_id,
    o.order_date,
    p.product_name,
    p.price,
    od.quantity
FROM orders o
JOIN order_details od ON o.order_id = od.order_id
JOIN products p ON od.product_id = p.product_id
WHERE o.order_date BETWEEN '2023-01-01' AND '2023-01-31 23:59:59';
```

**方法二：提取月份过滤**  
```sql
WHERE EXTRACT(YEAR FROM o.order_date) = 2023 
  AND EXTRACT(MONTH FROM o.order_date) = 1
```

**核心要点**：  
1. 使用`JOIN`关联三表：订单表、订单明细表、商品表  
2. 通过`WHERE`过滤时间范围，注意包含月末最后一秒  
3. 选择关键字段：订单ID、下单时间、商品名、单价、购买数量  

**注意事项**：  
- 建议对`order_date`字段建立索引提升查询性能  
- 若需商品分类信息，可继续关联商品分类表  
- 实际场景中应避免`SELECT *`，明确指定需要的字段

---

## 15. 编写 SQL，查询在 2023 年 3 月份的所有订单及其对应客户的姓名

🟡 **中等** | 标签：SQL, SQL进阶, SQL电商场景

假设数据库中包含订单表（orders）和客户表（customers），结构如下：
- **orders**：order_id（订单ID）, customer_id（客户ID）, order_date（下单日期）
- **customers**：customer_id（客户ID）, customer_name（客户姓名）

查询2023年3月所有订单及对应客户姓名的SQL语句如下：
```sql
SELECT 
    o.order_id,
    o.order_date,
    c.customer_name
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date BETWEEN '2023-03-01' AND '2023-03-31';
```

**核心要点说明：**
1. **表连接**：使用`INNER JOIN`关联订单表与客户表，通过`customer_id`建立连接，确保只返回有对应客户的订单。
2. **日期筛选**：`WHERE`子句通过`BETWEEN`指定2023年3月的日期范围（包含首尾日期），也可替换为`YEAR(o.order_date) = 2023 AND MONTH(o.order_date) = 3`，但`BETWEEN`通常效率更高。
3. **字段选择**：选择订单ID、下单日期和客户姓名，避免使用`SELECT *`以提高查询性能与可读性。

此查询能准确获取目标数据，若表数据量大，建议在`order_date`和`customer_id`字段上建立索引以优化性能。

---

## 16. 编写 SQL，查询总金额大于 100 的订单及其对应客户的姓名

🟡 **中等** | 标签：SQL, SQL进阶, SQL电商场景

### 一、分析表结构与关联关系
根据电商场景，通常存在以下表：
1.  `customers`（客户表）：含 `customer_id`（主键）、`name`（姓名）。
2.  `orders`（订单主表）：含 `order_id`（主键）、`customer_id`（外键，关联客户）。
3.  `order_details`（订单明细表）：含 `order_id`（外键）、`product_id`、`quantity`（数量）、`unit_price`（单价）。

核心需求：订单总金额 = SUM(数量 × 单价)，且 > 100，并返回对应客户姓名。

### 二、SQL查询语句
以下提供两种常用写法：

**写法一：使用子查询（先聚合计算，再关联）**
```sql
SELECT o.order_id, 
       c.name AS customer_name,
       sub.total_amount
FROM orders o
JOIN (
    SELECT order_id, 
           SUM(quantity * unit_price) AS total_amount
    FROM order_details
    GROUP BY order_id
    HAVING SUM(quantity * unit_price) > 100
) sub ON o.order_id = sub.order_id
JOIN customers c ON o.customer_id = c.customer_id;
```

**写法二：使用多表直接连接与聚合（更常用）**
```sql
SELECT o.order_id,
       c.name AS customer_name,
       SUM(d.quantity * d.unit_price) AS total_amount
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_details d ON o.order_id = d.order_id
GROUP BY o.order_id, c.name
HAVING SUM(d.quantity * d.unit_price) > 100;
```

### 三、要点总结
1.  **核心逻辑**：通过 `orders` 表关联 `customers` 和 `order_details` 表，计算订单总金额并筛选。
2.  **关键函数**：使用 `SUM()` 进行聚合计算，`GROUP BY` 按订单分组。
3.  **过滤条件**：使用 `HAVING` 对聚合后的结果（总金额）进行过滤

---

## 17. 编写 SQL，查询所有订单的总商品数量和总金额，并按订单日期升序排序

🟡 **中等** | 标签：SQL, SQL进阶, SQL电商场景

```sql
SELECT 
    o.order_id,
    o.order_date,
    SUM(oi.quantity) AS total_quantity,
    SUM(oi.quantity * oi.unit_price) AS total_amount
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY o.order_id, o.order_date
ORDER BY o.order_date ASC;
```

**核心要点说明：**

1. **多表关联**：通过订单主表`orders`和订单明细表`order_items`进行`JOIN`连接，基于订单ID关联商品数据。

2. **聚合计算**：
   - `SUM(oi.quantity)`：汇总每个订单中所有商品的**总数量**
   - `SUM(oi.quantity * oi.unit_price)`：计算每个订单的**总金额**（数量×单价）

3. **分组与排序**：
   - `GROUP BY`：按订单ID和日期分组，确保每个订单独立计算
   - `ORDER BY o.order_date ASC`：按订单日期升序排列（ASC可省略）

**注意事项**：
- 需确保关联字段`order_id`在两表中存在且数据一致
- 若订单表已有金额字段，可直接求和避免重复计算
- 实际场景中需考虑索引优化（如`order_date`索引）

---

## 18. 编写 SQL，查询有订单记录的客户姓名及其最新的订单日期

🟡 **中等** | 标签：SQL, SQL进阶, SQL电商场景

为了查询有订单记录的客户姓名及其最新订单日期，假设数据库中有两个表：`customers`（客户表，包含`customer_id`和`customer_name`字段）和`orders`（订单表，包含`order_id`、`customer_id`和`order_date`字段）。核心思路是通过内连接关联两表，筛选出有订单的客户，再按客户分组并取最大订单日期。

具体步骤如下：
1. 使用`JOIN`将`customers`与`orders`按`customer_id`连接，确保只返回有订单的客户。
2. 按`customer_id`和`customer_name`分组，避免同名客户混淆。
3. 使用聚合函数`MAX(order_date)`获取每个客户的最新订单日期。
4. 为了结果清晰，可对列设置别名。

SQL代码如下：
```sql
SELECT 
    c.customer_name AS 客户姓名,
    MAX(o.order_date) AS 最新订单日期
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```

此查询满足基本需求，且效率较高。如果订单表数据量较大，可考虑在`customer_id`和`order_date`上建立索引以优化性能。此外，若客户姓名可能重复，按`customer_id`分组更安全。如果只需单个客户的最新订单，可添加

---

## 19. 编写 SQL，查询每个商品的销售总数量及其对应的订单总金额

🟡 **中等** | 标签：SQL, SQL基础, SQL电商场景

## 参考答案

### 思路分析

本题考察**GROUP BY分组聚合**的使用，核心是按商品维度汇总销售数据。通常涉及商品表和订单明细表的关联查询。

### SQL实现

```sql
SELECT 
    p.product_id,
    p.product_name,
    SUM(o.quantity) AS total_quantity,
    SUM(o.amount) AS total_amount
FROM products p
JOIN orders o ON p.product_id = o.product_id
GROUP BY p.product_id, p.product_name;
```

### 关键点说明

1. **JOIN关联**：通过`product_id`连接商品表与订单表，确保数据对应正确

2. **GROUP BY分组**：按商品ID和名称分组，实现每个商品独立统计

3. **SUM聚合函数**：分别计算数量（`quantity`）和金额（`amount`）的总和

### 扩展考虑

- **过滤条件**：可加`WHERE`限定时间范围，如`WHERE o.order_date BETWEEN '2024-01-01' AND '2024-12-31'`
- **排序优化**：加`ORDER BY total_amount DESC`按金额降序排列
- **空值处理**：使用`COALESCE(SUM(amount), 0)`避免NULL显示
- **LEFT JOIN**：若需展示无销量的商品，改为左连接

### 涉及知识点

本题覆盖**表连接、分组聚合、聚合函数**三个核心SQL技能，是电商数据分析的基础场景。

---

## 20. 编写 SQL，查询每个客户的姓名及其第一个订单的总金额

🟡 **中等** | 标签：SQL, SQL进阶, SQL电商场景

**问题分析**：本题考察窗口函数或子查询的应用，需关联客户、订单及订单详情表。关键点在于准确识别“第一个订单”，通常基于下单时间排序，并计算该订单的总金额。

**SQL实现示例**：
```sql
SELECT
    c.customer_name,
    o.order_amount AS first_order_amount
FROM
    customers c
    JOIN (
        SELECT
            customer_id,
            order_amount,
            ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_time) AS rn
        FROM
            orders
    ) o ON c.customer_id = o.customer_id
WHERE o.rn = 1;
```
**关键点说明**：
1. **表关联**：客户表（customers）与订单表（orders）通过客户ID关联。
2. **窗口函数**：使用 `ROW_NUMBER()` 按客户分组并按订单时间排序，标记每个客户的首单。
3. **子查询筛选**：外层查询通过 `rn=1` 过滤出每个客户的首单记录。
4. **扩展性**：若需订单总金额来自多表，可在子查询中提前关联聚合计算。

**注意事项**：实际表结构可能差异（如金额需关联明细表），需根据数据库设计调整。此解法效率较高，适合电商场景大数据量处理。

---

## 21. 编写 SQL，查询每个客户的订单总数量和总金额，并按总金额降序排序

🟡 **中等** | 标签：SQL, SQL进阶, SQL电商场景

```sql
SELECT
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) AS total_orders,
    SUM(o.total_amount) AS total_amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name
ORDER BY total_amount DESC;
```

**答案要点：**
1. **多表连接**：使用 `LEFT JOIN` 关联客户表与订单表，确保无订单的客户也能显示。
2. **分组聚合**：按客户ID和姓名分组，使用 `COUNT` 统计订单数量，`SUM` 计算订单总金额。
3. **排序逻辑**：通过 `ORDER BY total_amount DESC` 实现按总金额降序排列。
4. **字段选择**：查询结果包含客户标识信息及聚合后的统计指标，符合题目要求。

此SQL实现了客户维度的订单汇总分析，适用于电商场景中的客户消费行为评估。

---

## 22. 编写 SQL，查询每个订单的总金额及其包含的商品总数量

🟡 **中等** | 标签：SQL, SQL进阶, SQL电商场景

```sql
SELECT
    o.order_id AS 订单ID,
    SUM(d.quantity * d.unit_price) AS 订单总金额,
    SUM(d.quantity) AS 商品总数量
FROM
    orders o
JOIN
    order_details d ON o.order_id = d.order_id
GROUP BY
    o.order_id;
```

**核心逻辑解析：**

1.  **关联查询**：使用 `JOIN` 将订单主表 (`orders`) 与订单明细表 (`order_details`) 通过 `order_id` 关联，以获取订单下所有商品的信息。
2.  **聚合计算**：
    *   **订单总金额**：需计算每件商品的 `小计金额 (数量 * 单价)`，再对同一订单下的所有小计金额进行 `SUM` 求和。
    *   **商品总数量**：直接对明细表中的 `quantity` 字段进行 `SUM` 求和，得到该订单所有商品的累计数量。
3.  **分组**：使用 `GROUP BY o.order_id` 确保上述 `SUM` 函数是针对每个独立的订单进行计算。

**关键点**：理解“总金额”需要分两步计算（先乘后和），而“总数量”直接求和即可。此查询展示了 `JOIN`、聚合函数 (`SUM`) 与分组 (`GROUP BY`) 的核心用法。实际字段名需根据具体表结构调整。

---

## 23. 编写 SQL，查询每个订单的订单日期，并计算订单日期的年份和季度

🟡 **中等** | 标签：SQL, SQL进阶, SQL电商场景

### 解题思路
本题考察在电商场景下对日期数据的处理能力。核心是使用SQL的日期函数从订单日期中提取年份和季度信息，便于进行时间维度的业务分析（如按季度统计销售额）。

### 参考答案
```sql
SELECT 
    order_id,
    order_date,
    YEAR(order_date) AS order_year,
    QUARTER(order_date) AS order_quarter
FROM orders;
```

**说明：**
1.  **核心函数**：使用 `YEAR()` 和 `QUARTER()` 函数分别提取订单日期的年份和季度。这是MySQL和SQL Server的标准语法。
2.  **结果展示**：查询结果将包含四列：订单ID、原始订单日期、计算得到的年份（如2023）和季度（取值为1,2,3,4）。
3.  **语法差异**：在PostgreSQL中，需使用 `EXTRACT(YEAR FROM order_date)` 和 `EXTRACT(QUARTER FROM order_date)`。面试时可主动说明对不同数据库语法的了解，这是加分项。
4.  **业务意义**：此查询是进行时间序列分析（如季节性销售趋势）的基础步骤，为后续的GROUP BY聚合统计做好了数据准备。

---

## 24. 编写 SQL，查询每个订单的详细商品信息，包括商品 ID、数量、价格和总金额

🟡 **中等** | 标签：SQL, SQL基础, SQL电商场景

```sql
SELECT 
    o.order_id AS 订单ID,
    d.product_id AS 商品ID,
    d.quantity AS 数量,
    d.unit_price AS 单价,
    SUM(d.quantity * d.unit_price) AS 订单总金额
FROM orders o
JOIN order_details d ON o.order_id = d.order_id
GROUP BY o.order_id, d.product_id, d.quantity, d.unit_price
ORDER BY o.order_id;
```

**核心要点解析：**
1.  **多表连接**：使用 `JOIN` 关联订单主表 (`orders`) 与订单明细表 (`order_details`)，通过 `order_id` 进行连接。
2.  **字段计算**：总金额由明细中的 `数量 * 单价` 计算得出，使用 `SUM()` 进行聚合。
3.  **分组与聚合**：`GROUP BY` 子句按订单ID、商品ID等维度进行分组，确保每个订单中每个商品的详细信息独立呈现。
4.  **排序优化**：`ORDER BY` 按订单ID排序，便于结果查阅。

**面试场景提示：** 实际回答时可补充说明，若需按订单级汇总总金额，可在外层嵌套查询或调整聚合逻辑。此查询假设表结构基于常见电商设计，若表名或字段名不同需相应调整。

---

## 25. 编写 SQL，查询至少有一个订单金额超过 200 的客户姓名

🟡 **中等** | 标签：SQL, SQL进阶, SQL电商场景

## 解题思路

**假定表结构：**
- `customers` 表：`customer_id`（客户ID）、`customer_name`（客户姓名）
- `orders` 表：`order_id`（订单ID）、`customer_id`（客户ID）、`amount`（订单金额）

---

## 方法一：EXISTS 子查询（推荐）

```sql
SELECT customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1 
    FROM orders o 
    WHERE o.customer_id = c.customer_id 
      AND o.amount > 200
);
```

**优点**：EXISTS 找到一条匹配记录即返回，效率较高，适合大数据量场景。

---

## 方法二：JOIN + DISTINCT

```sql
SELECT DISTINCT c.customer_name
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.amount > 200;
```

**优点**：写法直观，易理解；需用 DISTINCT 去重（一个客户可能有多个超200订单）。

---

## 方法三：IN 子查询

```sql
SELECT customer_name
FROM customers
WHERE customer_id IN (
    SELECT DISTINCT customer_id 
    FROM orders 
    WHERE amount > 200
);
```

**优点**：逻辑清晰；当子查询结果集较大时性能可能不如 EXISTS。

---

## 核心要点总结

| 考点 | 说明 |
|------|------|
| 多表关联 | 客户表与订单表通过 `customer_id` 连接 |
| 聚合筛选 | 不需要 GROUP BY，只需存在性判断 |
| 去重处理 | 使用 EXISTS 或 DISTINCT 避免重复 |
| 性能考量 | EXISTS 通常优于 IN（尤其是大子查询集） |

**面试加分点**：主动说明三种写法的性能差异，体现对 SQL 执行原理的理解。

---

## 26. 编写 SQL，查询订单总金额最高的客户姓名和对应的订单总金额

🟡 **中等** | 标签：SQL, SQL进阶, SQL电商场景

**方法一：使用排序和限制**

```sql
SELECT 
    c.customer_name,
    SUM(o.order_amount) AS total_amount
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_name
ORDER BY total_amount DESC
LIMIT 1;
```

**方法二：使用子查询匹配最大值**

```sql
SELECT 
    customer_name,
    total_amount
FROM (
    SELECT 
        c.customer_name,
        SUM(o.order_amount) AS total_amount
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_name
) AS customer_totals
WHERE total_amount = (
    SELECT SUM(order_amount)
    FROM orders
    GROUP BY customer_id
    ORDER BY SUM(order_amount) DESC
    LIMIT 1
);
```

**核心要点：**
1. 使用 `JOIN` 关联客户表与订单表
2. 通过 `GROUP BY` 按客户分组聚合计算总金额
3. 方法一直接排序取第一条高效简洁
4. 方法二先计算最大金额再匹配，能处理并列最高情况
5. 实际应用中需考虑表字段命名和索引优化

---

## 27. 编写 SQL，查询订单数量最多的客户姓名和订单数量

🟡 **中等** | 标签：SQL, SQL进阶, SQL电商场景

```sql
SELECT c.customer_name, COUNT(o.order_id) AS order_count
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name
HAVING COUNT(o.order_id) = (
    SELECT MAX(order_cnt) FROM (
        SELECT customer_id, COUNT(order_id) AS order_cnt
        FROM orders
        GROUP BY customer_id
    ) AS temp
);
```

**解题思路：**

1.  **连接与分组**：首先通过 `JOIN` 关联 `customers` 和 `orders` 表，使用 `GROUP BY` 按客户分组。
2.  **计算数量**：对每组使用 `COUNT(o.order_id)` 计算每个客户的订单总数。
3.  **筛选最大值**：使用 `HAVING` 子句进行条件筛选。核心是通过一个子查询，先计算出所有客户的订单数，再用 `MAX()` 函数找到其中的最大值。
4.  **处理并列**：此查询可以正确返回订单数并列第一的所有客户。

**优化说明**：在面试中，使用子查询或窗口函数（如 `RANK()`）均可。此方案展示了经典解法，逻辑清晰，能有效应对“找最值”的场景。

---

## 28. 编写 SQL，查询订单数量超过 5 的每个客户的订单数量

🟡 **中等** | 标签：SQL, SQL进阶, SQL电商场景

```sql
SELECT
    customer_id,
    COUNT(order_id) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(order_id) > 5;
```

**解题思路说明：**

1.  **需求理解**：目标是筛选出那些总订单数大于5的客户，并展示其客户ID和对应的订单总数。
2.  **核心逻辑**：
    *   使用 `GROUP BY customer_id` 将数据按客户分组，以便统计每位客户的订单数量。
    *   使用聚合函数 `COUNT(order_id)` 计算每个分组（即每个客户）中的订单总数。
3.  **过滤条件**：关键点在于使用 `HAVING` 子句对分组后的结果进行过滤，筛选出 `COUNT(order_id) > 5` 的组（即订单数超过5的客户）。这里必须用 `HAVING` 而非 `WHERE`，因为过滤条件作用于聚合函数的结果。
4.  **结果**：查询将返回一个结果集，包含满足条件的 `customer_id` 和他们对应的 `order_count`。

---

## 29. 编写 SQL，统计每个客户的订单总金额

🟡 **中等** | 标签：SQL, SQL进阶, SQL电商场景

```sql
SELECT
    c.customer_id,
    c.customer_name,
    SUM(o.order_amount) AS total_amount
FROM
    customers c
LEFT JOIN
    orders o ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id, c.customer_name
ORDER BY
    total_amount DESC;
```

**核心要点说明：**

1. **主表选择**：以客户表为主表进行 `LEFT JOIN`，确保无订单的客户也能显示（金额为0或NULL）。
2. **聚合逻辑**：使用 `SUM()` 对订单金额进行求和，按客户ID和姓名分组（通常客户ID为主键，姓名可省略）。
3. **结果排序**：按总金额降序排列，便于识别高价值客户。
4. **业务考虑**：实际场景中可增加条件过滤，如只统计已完成订单（添加 `WHERE o.status = 'completed'`），或处理金额精度（使用 `ROUND()` 或 `DECIMAL` 类型）。
5. **性能提示**：建议在 `customer_id` 和 `order_amount` 字段建立索引以提升查询效率。

---

## 30. 编写 SQL，查询所有订单在每个客户订单中的累计总金额，并按订单日期升序排序

🔴 **困难** | 标签：SQL, SQL进阶, SQL电商场景

```sql
SELECT 
    customer_id,
    order_id,
    order_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_total
FROM orders
ORDER BY customer_id, order_date;
```

**说明：**
1. 使用窗口函数 `SUM() OVER()` 实现累计计算
2. `PARTITION BY customer_id` 按客户分区计算累计值
3. `ORDER BY order_date` 按订单日期排序
4. `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` 明确计算范围为从分区开始到当前行
5. 结果先按客户ID、再按订单日期升序排列，符合题目要求

---

## 31. 编写 SQL，查询每个客户的姓名及其上一个订单的总金额

🔴 **困难** | 标签：SQL, SQL进阶, SQL电商场景

这是一个常见的SQL面试题，考察窗口函数和关联查询的应用。假设数据库中存在两张表：Customers（包含customer_id和name字段）和Orders（包含order_id、customer_id、order_date和total_amount字段）。

解决方案是使用窗口函数ROW_NUMBER()，为每个客户的订单按日期降序排序，然后提取每个客户排名为1的订单（即最新的订单）。具体步骤如下：
1. 在Orders表上，使用ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) 为每个客户的订单分配排名，排名为1的行即为“上一个订单”。
2. 将排名结果作为子查询，与Customers表通过customer_id关联，以获取客户姓名。
3. 从关联结果中选取客户姓名和对应订单的总金额。

以下是完整的SQL查询示例：
```sql
WITH RankedOrders AS (
    SELECT 
        customer_id, 
        total_amount,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rn
    FROM Orders
)
SELECT 
    c.name AS 客户姓名,
    ro.total_amount AS 上一个订单总金额
FROM Customers c
JOIN RankedOrders ro ON c.customer_id = ro.customer_id AND ro.rn = 1;
```
此查询高效地解决了问题：窗口函数避免了复杂自连接，适合处理大数据量；CTE使结构清晰易读。在电商场景中，此方法可用于客户行为分析、推荐系统

---

## 32. 编写 SQL，查询每个客户的最新订单总金额及其在客户所有订单总金额中的比例

🔴 **困难** | 标签：SQL, SQL进阶, SQL电商场景

```sql
SELECT 
    c.customer_id,
    c.customer_name,
    o.order_date AS latest_order_date,
    od.total_amount AS latest_order_amount,
    od.total_amount / SUM(od.total_amount) OVER (PARTITION BY c.customer_id) AS amount_ratio
FROM 
    customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN (
    SELECT 
        order_id, 
        SUM(quantity * unit_price) AS total_amount
    FROM order_details
    GROUP BY order_id
) od ON o.order_id = od.order_id
WHERE 
    o.order_date = (
        SELECT MAX(order_date) 
        FROM orders 
        WHERE customer_id = c.customer_id
    )
ORDER BY 
    c.customer_id;
```

**思路说明：**
1. **核心逻辑**：使用子查询定位每个客户的最新订单（通过MAX(order_date)筛选），再关联订单详情计算该订单总金额。
2. **比例计算**：通过窗口函数SUM() OVER(PARTITION BY)计算客户所有订单的总金额，再用最新订单金额除以总金额得到比例。
3. **优化提示**：实际场景建议为orders表的(customer_id, order_date)建立索引提升性能。
4. **示例数据**：若客户A有订单金额分别为100、200、150，最新订单200元，则比例=200/(100+200+150)=44.4%。

**注意事项**：若最新订单有多笔（同日），需明确业务规则（如取金额最大或订单号最大），可通过ROW_NUMBER()窗口函数进一步优化。

---

## 33. 编写 SQL，查询每个订单的总商品数量及其包含的商品列表（用逗号分隔）

🔴 **困难** | 标签：SQL, SQL进阶, SQL电商场景

查询每个订单的总商品数量和商品列表是电商数据分析中的常见需求，通常涉及订单表和订单项表。假设数据库包含`orders`表（订单信息）和`order_items`表（订单商品明细，包含`order_id`、`product_id`和`quantity`等字段），并关联`products`表获取商品名称。以下是基于MySQL的SQL实现，核心是使用`GROUP BY`分组和聚合函数。

SQL查询代码如下：
```sql
SELECT 
    o.order_id AS 订单号,
    SUM(oi.quantity) AS 总商品数量,
    GROUP_CONCAT(p.product_name SEPARATOR ', ') AS 商品列表
FROM 
    orders o
JOIN 
    order_items oi ON o.order_id = oi.order_id
JOIN 
    products p ON oi.product_id = p.product

---

## 34. 编写 SQL，查询订单总金额超过其客户所有订单平均金额的订单

🔴 **困难** | 标签：SQL, SQL进阶, SQL电商场景

**思路分析：**  
需分两步：①计算每个客户的平均订单金额；②将订单金额与对应客户的平均值比较。可通过关联子查询或窗口函数实现。

**SQL示例（关联子查询）：**  
```sql
SELECT o.order_id, o.amount, o.customer_id
FROM orders o
WHERE o.amount > (
    SELECT AVG(amount)
    FROM orders
    WHERE customer_id = o.customer_id
);
```

**SQL示例（窗口函数）：**  
```sql
SELECT order_id, amount, customer_id
FROM (
    SELECT *,
           AVG(amount) OVER (PARTITION BY customer_id) AS avg_amount
    FROM orders
) t
WHERE amount > avg_amount;
```

**关键点：**  
1. 子查询中需用 `WHERE` 条件关联同一客户；  
2. 窗口函数 `PARTITION BY customer_id` 实现分组平均；  
3. 结果需包含订单ID、金额、客户ID等业务字段。

---

