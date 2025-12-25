# 07-WHERE条件查询

> 掌握MySQL的WHERE条件查询，灵活筛选数据

---

## 📖 本章目标

- 掌握比较运算符的使用
- 掌握逻辑运算符的使用
- 掌握LIKE模糊查询
- 掌握NULL值的处理
- 掌握IN和NOT IN的使用
- 理解查询优化原则

---

## 一、比较运算符

### 1.1 等值查询（=）

```sql
-- 单条件
SELECT * FROM users WHERE id = 1;
SELECT * FROM users WHERE username = 'zhangsan';
SELECT * FROM users WHERE status = 1;

-- 字符串比较（大小写）
-- 默认：不区分大小写（utf8mb4_unicode_ci）
SELECT * FROM users WHERE username = 'ZhangSan';  -- 能查到'zhangsan'

-- 区分大小写查询（使用BINARY）
SELECT * FROM users WHERE BINARY username = 'zhangsan';

-- 或创建表时使用_bin排序规则
CREATE TABLE users (
    username VARCHAR(50) NOT NULL COLLATE utf8mb4_bin
) ENGINE=InnoDB;
```

### 1.2 不等查询（!=、<>）

```sql
-- 两种写法等价
SELECT * FROM users WHERE status != 1;
SELECT * FROM users WHERE status <> 1;

-- 查询非指定值
SELECT * FROM products WHERE category_id != 1;
SELECT * FROM orders WHERE status <> 5;  -- 状态不是已取消

-- 注意：不等查询不包括NULL
SELECT * FROM users WHERE deleted_at != NULL;  -- ❌ 错误，查不到数据
SELECT * FROM users WHERE deleted_at IS NOT NULL;  -- ✅ 正确
```

### 1.3 范围查询（>、<、>=、<=）

```sql
-- 大于
SELECT * FROM products WHERE price_cents > 10000;  -- 价格大于100元
SELECT * FROM users WHERE created_at > '2024-01-01';

-- 小于
SELECT * FROM products WHERE stock < 10;  -- 库存少于10
SELECT * FROM users WHERE age < 18;

-- 大于等于
SELECT * FROM orders WHERE total_amount >= 10000;

-- 小于等于
SELECT * FROM products WHERE price_cents <= 50000;

-- 组合范围查询
SELECT * FROM products
WHERE price_cents >= 10000 AND price_cents <= 50000;
```

### 1.4 区间查询（BETWEEN ... AND）

```sql
-- BETWEEN包含边界值
SELECT * FROM products
WHERE price_cents BETWEEN 10000 AND 50000;
-- 等价于
SELECT * FROM products
WHERE price_cents >= 10000 AND price_cents <= 50000;

-- 时间范围查询
SELECT * FROM orders
WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31';

-- 注意：BETWEEN A AND B，A必须小于等于B
SELECT * FROM products
WHERE price_cents BETWEEN 50000 AND 10000;  -- ❌ 错误，查不到数据

-- NOT BETWEEN
SELECT * FROM products
WHERE price_cents NOT BETWEEN 10000 AND 50000;
-- 等价于
SELECT * FROM products
WHERE price_cents < 10000 OR price_cents > 50000;
```

---

## 二、逻辑运算符

### 2.1 AND（与）

```sql
-- 两个条件都满足
SELECT * FROM users
WHERE status = 1 AND deleted_at IS NULL;

-- 多个条件
SELECT * FROM products
WHERE status = 1
AND stock > 0
AND price_cents >= 10000
AND category_id = 1;

-- AND优先级高于OR
SELECT * FROM products
WHERE category_id = 1 AND status = 1 OR category_id = 2;
-- 等价于
SELECT * FROM products
WHERE (category_id = 1 AND status = 1) OR category_id = 2;
```

### 2.2 OR（或）

```sql
-- 任一条件满足
SELECT * FROM users
WHERE status = 1 OR status = 2;

-- 多个OR条件
SELECT * FROM products
WHERE category_id = 1
OR category_id = 2
OR category_id = 3;

-- 使用括号明确优先级
SELECT * FROM products
WHERE (category_id = 1 OR category_id = 2)
AND status = 1;
```

### 2.3 NOT（非）

```sql
-- 取反
SELECT * FROM users WHERE NOT status = 1;
-- 等价于
SELECT * FROM users WHERE status != 1;

-- NOT IN
SELECT * FROM users WHERE id NOT IN (1, 2, 3);

-- NOT LIKE
SELECT * FROM users WHERE username NOT LIKE 'admin%';

-- NOT BETWEEN
SELECT * FROM products WHERE price_cents NOT BETWEEN 10000 AND 50000;
```

### 2.4 优先级与括号

```sql
-- 优先级：NOT > AND > OR

-- 示例1：默认优先级
SELECT * FROM products
WHERE category_id = 1 AND status = 1 OR category_id = 2;
-- 解析为：(category_id = 1 AND status = 1) OR category_id = 2

-- 示例2：使用括号改变优先级
SELECT * FROM products
WHERE category_id = 1 AND (status = 1 OR category_id = 2);

-- 推荐：使用括号明确表达意图
SELECT * FROM products
WHERE (category_id = 1 OR category_id = 2)
AND status = 1
AND stock > 0;
```

---

## 三、LIKE模糊查询

### 3.1 LIKE基础

```sql
-- % 通配符：匹配任意字符（0个或多个）
SELECT * FROM users WHERE username LIKE 'zhang%';  -- zhang开头
SELECT * FROM users WHERE username LIKE '%san';    -- san结尾
SELECT * FROM users WHERE username LIKE '%ang%';   -- 包含ang

-- _ 通配符：匹配单个字符
SELECT * FROM users WHERE username LIKE 'user_';  -- user + 1个字符
SELECT * FROM users WHERE username LIKE 'user__'; -- user + 2个字符

-- 组合使用
SELECT * FROM users WHERE username LIKE 'z%san';  -- z开头，san结尾
SELECT * FROM users WHERE username LIKE 'user_%'; -- user开头，至少6个字符
```

### 3.2 LIKE性能问题

```sql
-- ❌ 前导模糊查询：性能差，无法使用索引
SELECT * FROM users WHERE username LIKE '%san';
SELECT * FROM users WHERE username LIKE '%ang%';

-- ✅ 前缀匹配：性能好，可以使用索引
SELECT * FROM users WHERE username LIKE 'zhang%';

-- 查看执行计划
EXPLAIN SELECT * FROM users WHERE username LIKE '%san';
-- type: ALL（全表扫描）

EXPLAIN SELECT * FROM users WHERE username LIKE 'zhang%';
-- type: range（索引范围扫描）
```

### 3.3 转义字符

```sql
-- 查询包含 % 或 _ 的字符串
SELECT * FROM articles WHERE title LIKE '%\%%';  -- 包含%
SELECT * FROM articles WHERE title LIKE '%\_%';  -- 包含_

-- 使用ESCAPE指定转义字符
SELECT * FROM articles WHERE title LIKE '%#%%' ESCAPE '#';  -- 包含%
SELECT * FROM articles WHERE title LIKE '%#_%' ESCAPE '#';  -- 包含_
```

### 3.4 全文索引（FULLTEXT）

```sql
-- 创建全文索引
CREATE TABLE articles (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    content TEXT NOT NULL,

    FULLTEXT INDEX ft_title_content (title, content) WITH PARSER ngram
) ENGINE=InnoDB;

-- 全文搜索（比LIKE快）
SELECT * FROM articles
WHERE MATCH(title, content) AGAINST('MySQL教程');

-- 布尔模式搜索
SELECT * FROM articles
WHERE MATCH(title, content) AGAINST('+MySQL -Oracle' IN BOOLEAN MODE);
-- +MySQL：必须包含
-- -Oracle：不包含

-- 查询模式搜索（支持通配符）
SELECT * FROM articles
WHERE MATCH(title, content) AGAINST('MySQL*' IN BOOLEAN MODE);
```

---

## 四、NULL值处理

### 4.1 IS NULL / IS NOT NULL

```sql
-- 查询NULL值
SELECT * FROM users WHERE deleted_at IS NULL;      -- 未删除
SELECT * FROM users WHERE phone IS NULL;           -- 未填写手机号

-- 查询非NULL值
SELECT * FROM users WHERE deleted_at IS NOT NULL;  -- 已删除
SELECT * FROM users WHERE phone IS NOT NULL;       -- 已填写手机号

-- ❌ 错误用法：不能用 = 或 != 查询NULL
SELECT * FROM users WHERE deleted_at = NULL;   -- 查不到数据
SELECT * FROM users WHERE deleted_at != NULL;  -- 查不到数据
```

### 4.2 NULL的比较陷阱

```sql
-- NULL与任何值比较都是NULL（不是TRUE也不是FALSE）
SELECT NULL = NULL;    -- NULL
SELECT NULL != NULL;   -- NULL
SELECT NULL > 1;       -- NULL
SELECT 1 IN (1, 2, NULL);  -- 1（1在列表中）
SELECT 3 IN (1, 2, NULL);  -- NULL（3不在列表中，但有NULL）

-- NULL与逻辑运算
SELECT NULL AND TRUE;   -- NULL
SELECT NULL AND FALSE;  -- FALSE
SELECT NULL OR TRUE;    -- TRUE
SELECT NULL OR FALSE;   -- NULL
```

### 4.3 IFNULL函数

```sql
-- IFNULL(expr, value)：如果expr为NULL，返回value
SELECT
    id,
    username,
    IFNULL(phone, '未填写') AS phone,
    IFNULL(nickname, username) AS display_name
FROM users;

-- 示例：统计平均年龄（忽略NULL）
SELECT AVG(IFNULL(age, 0)) AS avg_age FROM users;

-- 示例：排序时NULL值处理
SELECT * FROM users
ORDER BY IFNULL(deleted_at, '9999-12-31') ASC;
-- NULL排在最后
```

### 4.4 COALESCE函数

```sql
-- COALESCE(expr1, expr2, expr3, ...)：返回第一个非NULL值
SELECT COALESCE(NULL, NULL, 'value', NULL);  -- 'value'

SELECT
    id,
    username,
    COALESCE(nickname, username, '匿名用户') AS display_name
FROM users;

-- 示例：优先使用手机号，否则邮箱
SELECT
    id,
    username,
    COALESCE(phone, email, '无联系方式') AS contact
FROM users;
```

### 4.5 NULLIF函数

```sql
-- NULLIF(expr1, expr2)：如果expr1 = expr2，返回NULL，否则返回expr1
SELECT NULLIF(1, 1);  -- NULL
SELECT NULLIF(1, 2);  -- 1

-- 示例：避免除零错误
SELECT
    product_id,
    sales_count,
    view_count,
    sales_count / NULLIF(view_count, 0) AS conversion_rate
FROM products;
-- view_count=0时，返回NULL而不是报错
```

---

## 五、IN和NOT IN

### 5.1 IN基础

```sql
-- IN：匹配列表中的任意值
SELECT * FROM users WHERE id IN (1, 2, 3, 4, 5);

-- 等价于
SELECT * FROM users
WHERE id = 1 OR id = 2 OR id = 3 OR id = 4 OR id = 5;

-- 字符串IN
SELECT * FROM users WHERE status IN (1, 2);
SELECT * FROM users WHERE username IN ('zhangsan', 'lisi', 'wangwu');

-- 子查询IN
SELECT * FROM orders
WHERE user_id IN (
    SELECT id FROM users WHERE status = 1
);
```

### 5.2 IN vs OR性能

```sql
-- IN性能通常优于多个OR
-- ✅ 推荐
SELECT * FROM users WHERE id IN (1, 2, 3, 4, 5);

-- ❌ 不推荐
SELECT * FROM users WHERE id = 1 OR id = 2 OR id = 3 OR id = 4 OR id = 5;

-- 但IN的值不宜过多（建议<1000）
-- ❌ 性能差
SELECT * FROM users WHERE id IN (1, 2, 3, ..., 10000);  -- 太多值
```

### 5.3 NOT IN及NULL陷阱

```sql
-- NOT IN：不匹配列表中的任意值
SELECT * FROM users WHERE id NOT IN (1, 2, 3);

-- ❌ NOT IN与NULL的陷阱
SELECT * FROM users WHERE id NOT IN (1, 2, NULL);
-- 查不到任何数据！

-- 原因：
-- id NOT IN (1, 2, NULL)
-- 等价于
-- id != 1 AND id != 2 AND id != NULL
-- id != NULL 总是NULL（不是TRUE）
-- 所以整个表达式是 TRUE AND TRUE AND NULL = NULL

-- 解决方案：过滤NULL
SELECT * FROM users WHERE id NOT IN (
    SELECT IFNULL(user_id, -1) FROM blacklist
);

-- 或使用NOT EXISTS（推荐）
SELECT * FROM users u
WHERE NOT EXISTS (
    SELECT 1 FROM blacklist b WHERE b.user_id = u.id
);
```

---

## 六、其他常用条件

### 6.1 EXISTS / NOT EXISTS

```sql
-- EXISTS：子查询至少返回一行，返回TRUE
SELECT * FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id
);
-- 查询有订单的用户

-- NOT EXISTS：子查询不返回任何行，返回TRUE
SELECT * FROM users u
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id
);
-- 查询没有订单的用户

-- EXISTS vs IN性能对比
-- EXISTS：
-- 1. 外表驱动，逐行检查
-- 2. 找到就返回TRUE，短路
-- 3. 适合外表小、内表大

-- IN：
-- 1. 先执行子查询，生成结果集
-- 2. 外表每行与结果集匹配
-- 3. 适合外表大、内表小
```

### 6.2 REGEXP（正则表达式）

```sql
-- REGEXP：正则匹配
SELECT * FROM users WHERE username REGEXP '^zhang';  -- zhang开头
SELECT * FROM users WHERE email REGEXP '@qq\.com$';  -- @qq.com结尾

-- 示例：匹配手机号
SELECT * FROM users WHERE phone REGEXP '^1[3-9][0-9]{9}$';

-- 示例：匹配身份证号
SELECT * FROM users WHERE id_card REGEXP '^[1-9][0-9]{16}[0-9Xx]$';

-- NOT REGEXP
SELECT * FROM users WHERE username NOT REGEXP '^admin';
```

### 6.3 CASE WHEN（条件表达式）

```sql
-- 简单CASE表达式
SELECT
    id,
    username,
    status,
    CASE status
        WHEN 1 THEN '正常'
        WHEN 2 THEN '禁用'
        WHEN 3 THEN '注销'
        ELSE '未知'
    END AS status_text
FROM users;

-- 搜索CASE表达式
SELECT
    id,
    username,
    age,
    CASE
        WHEN age < 18 THEN '未成年'
        WHEN age >= 18 AND age < 60 THEN '成年'
        ELSE '老年'
    END AS age_group
FROM users;

-- CASE WHEN在WHERE中使用
SELECT * FROM products
WHERE
    CASE
        WHEN category_id = 1 THEN price_cents > 100000
        WHEN category_id = 2 THEN price_cents > 50000
        ELSE TRUE
    END;
```

---

## 七、时间日期查询

### 7.1 日期比较

```sql
-- 查询某天
SELECT * FROM orders WHERE DATE(created_at) = '2024-01-01';

-- 查询某月
SELECT * FROM orders WHERE DATE_FORMAT(created_at, '%Y-%m') = '2024-01';
SELECT * FROM orders WHERE created_at >= '2024-01-01' AND created_at < '2024-02-01';

-- 查询某年
SELECT * FROM orders WHERE YEAR(created_at) = 2024;
```

### 7.2 相对时间查询

```sql
-- 今天
SELECT * FROM orders WHERE DATE(created_at) = CURDATE();

-- 昨天
SELECT * FROM orders WHERE DATE(created_at) = DATE_SUB(CURDATE(), INTERVAL 1 DAY);

-- 本周
SELECT * FROM orders WHERE YEARWEEK(created_at) = YEARWEEK(NOW());

-- 本月
SELECT * FROM orders WHERE DATE_FORMAT(created_at, '%Y-%m') = DATE_FORMAT(NOW(), '%Y-%m');

-- 最近7天
SELECT * FROM orders WHERE created_at >= DATE_SUB(NOW(), INTERVAL 7 DAY);

-- 最近30天
SELECT * FROM orders WHERE created_at >= DATE_SUB(NOW(), INTERVAL 30 DAY);
```

### 7.3 时间范围查询

```sql
-- 时间范围
SELECT * FROM orders
WHERE created_at >= '2024-01-01 00:00:00'
AND created_at <= '2024-12-31 23:59:59';

-- 使用BETWEEN
SELECT * FROM orders
WHERE created_at BETWEEN '2024-01-01 00:00:00' AND '2024-12-31 23:59:59';

-- 注意：BETWEEN包含边界值
-- created_at = '2024-12-31 23:59:59' 会被包含
```

---

## 八、查询优化原则

### 8.1 索引使用原则

```sql
-- ✅ 能使用索引
SELECT * FROM users WHERE username = 'zhangsan';  -- username有索引
SELECT * FROM users WHERE username LIKE 'zhang%';  -- 前缀匹配

-- ❌ 不能使用索引
SELECT * FROM users WHERE YEAR(created_at) = 2024;  -- 函数操作
SELECT * FROM users WHERE username LIKE '%san';     -- 前导模糊
SELECT * FROM users WHERE id + 1 = 2;               -- 表达式操作
SELECT * FROM users WHERE id = '1';                 -- 隐式类型转换（假设id是INT）
```

### 8.2 避免SELECT *

```sql
-- ❌ 不推荐
SELECT * FROM users WHERE id = 1;

-- ✅ 推荐
SELECT id, username, email, created_at FROM users WHERE id = 1;

-- 原因：
-- 1. 返回不需要的字段，浪费带宽
-- 2. 无法使用覆盖索引
-- 3. 表结构变化时可能出问题
```

### 8.3 限制返回行数

```sql
-- 如果只需要判断是否存在，限制返回1行
SELECT id FROM users WHERE username = 'zhangsan' LIMIT 1;

-- 如果只需要几条数据，使用LIMIT
SELECT * FROM articles ORDER BY created_at DESC LIMIT 10;
```

---

## 九、实战案例

### 案例1：用户搜索

```sql
-- 按用户名或邮箱搜索
SELECT id, username, email, created_at
FROM users
WHERE (
    username LIKE CONCAT('%', :keyword, '%')
    OR email LIKE CONCAT('%', :keyword, '%')
)
AND status = 1
AND deleted_at IS NULL
ORDER BY created_at DESC
LIMIT 20;
```

### 案例2：商品筛选

```sql
-- 分类、价格区间、库存筛选
SELECT id, name, price_cents / 100 AS price, stock
FROM products
WHERE category_id = :category_id
AND price_cents BETWEEN :min_price AND :max_price
AND stock > 0
AND status = 1
AND deleted_at IS NULL
ORDER BY sales_count DESC
LIMIT 20 OFFSET :offset;
```

### 案例3：订单列表

```sql
-- 查询某用户的订单，支持状态筛选
SELECT
    order_no,
    total_amount / 100 AS total,
    status,
    created_at,
    paid_at
FROM orders
WHERE user_id = :user_id
AND (:status IS NULL OR status = :status)  -- 状态可选
AND deleted_at IS NULL
ORDER BY created_at DESC
LIMIT 20 OFFSET :offset;
```

---

## 十、本章总结

### 核心要点

1. **比较运算符**：
   - 等值：=
   - 不等：!=、<>
   - 范围：>、<、>=、<=
   - 区间：BETWEEN

2. **逻辑运算符**：
   - AND、OR、NOT
   - 使用括号明确优先级

3. **LIKE模糊查询**：
   - 前缀匹配可用索引
   - 前导模糊不可用索引
   - 考虑使用全文索引

4. **NULL值处理**：
   - IS NULL、IS NOT NULL
   - IFNULL、COALESCE
   - NOT IN的NULL陷阱

5. **IN和EXISTS**：
   - IN适合小结果集
   - EXISTS适合大结果集
   - 避免NOT IN + NULL

6. **优化原则**：
   - 避免函数操作字段
   - 避免前导模糊查询
   - 避免SELECT *
   - 使用LIMIT限制返回行数

### 下一步

完成本章学习后，你应该能够：
- ✅ 编写各种WHERE条件查询
- ✅ 正确处理NULL值
- ✅ 使用LIKE进行模糊查询
- ✅ 理解查询优化原则

下一节：[08-聚合与分组查询](./08-聚合与分组查询.md)
