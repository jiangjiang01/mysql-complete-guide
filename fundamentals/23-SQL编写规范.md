# 23-SQL编写规范

> 遵循SQL编写规范，提高代码质量和可维护性

---

## 📖 本章目标

- 掌握SQL命名规范
- 掌握SQL编写风格
- 学会编写可维护的SQL
- 避免常见SQL错误
- 掌握SQL安全最佳实践

---

## 一、命名规范

### 1.1 数据库命名

```sql
-- ✅ 推荐：小写字母 + 下划线
ecommerce_db
user_system
order_center

-- ❌ 不推荐：
EcommerceDB       -- 大写
ecommerce-db      -- 连字符
ecommerce.db      -- 点号
电商数据库         -- 中文
```

### 1.2 表命名

```sql
-- ✅ 推荐：
-- 1. 使用复数名词
users             -- 不是user
orders            -- 不是order
products          -- 不是product

-- 2. 使用下划线分隔
user_addresses
order_items
product_categories

-- 3. 关联表使用两个表名
user_roles        -- users和roles的关联表
article_tags      -- articles和tags的关联表

-- 4. 日志表、历史表使用后缀
login_logs        -- 登录日志
order_history     -- 订单历史
users_backup      -- 用户备份表

-- ❌ 不推荐：
tbl_users         -- 不要加tbl前缀
UserInfo          -- 不要使用驼峰
user-info         -- 不要使用连字符
t_user            -- 不要使用无意义前缀
```

### 1.3 字段命名

```sql
-- ✅ 推荐：
id                -- 主键
user_id           -- 外键，表名_id
username          -- 用户名
email             -- 邮箱
created_at        -- 创建时间
updated_at        -- 更新时间
deleted_at        -- 删除时间
is_active         -- 布尔字段，is_开头
status            -- 状态

-- ❌ 不推荐：
Id                -- 不要大写
userId            -- 不要驼峰
user_name         -- username更简洁
create_time       -- 使用created_at
active            -- 布尔字段用is_active
```

### 1.4 索引命名

```sql
-- ✅ 推荐：
-- 主键：pk_表名
PRIMARY KEY (id)  -- 自动命名为PRIMARY

-- 唯一索引：uniq_表名_字段名
UNIQUE KEY uniq_username (username)
UNIQUE KEY uniq_email (email)

-- 普通索引：idx_表名_字段名
INDEX idx_status (status)
INDEX idx_created_at (created_at)

-- 联合索引：idx_表名_字段1_字段2
INDEX idx_user_status_created (user_id, status, created_at)

-- 全文索引：ft_表名_字段名
FULLTEXT INDEX ft_title_content (title, content)

-- ❌ 不推荐：
INDEX index1 (username)        -- 无意义命名
INDEX user_index (username)    -- 不清晰
INDEX username_idx (username)  -- 顺序不对
```

### 1.5 约束命名

```sql
-- ✅ 推荐：
-- 外键：fk_从表_主表
CONSTRAINT fk_orders_users
    FOREIGN KEY (user_id) REFERENCES users(id)

CONSTRAINT fk_order_items_products
    FOREIGN KEY (product_id) REFERENCES products(id)

-- 检查约束：ck_表名_字段名
CONSTRAINT ck_users_age
    CHECK (age >= 0 AND age <= 150)

CONSTRAINT ck_orders_amount
    CHECK (total_amount >= 0)

-- ❌ 不推荐：
CONSTRAINT fk1 ...              -- 无意义命名
CONSTRAINT user_fk ...          -- 不清晰
```

---

## 二、SQL编写风格

### 2.1 大小写规范

```sql
-- ✅ 推荐：关键字大写，表名、字段名小写

-- SELECT语句
SELECT
    id,
    username,
    email,
    created_at
FROM users
WHERE status = 1
ORDER BY created_at DESC
LIMIT 10;

-- INSERT语句
INSERT INTO users (username, email, status)
VALUES ('zhangsan', 'zhangsan@example.com', 1);

-- UPDATE语句
UPDATE users
SET status = 1, updated_at = NOW()
WHERE id = 1;

-- DELETE语句
DELETE FROM users
WHERE id = 1;

-- ❌ 不推荐：全部小写或全部大写
select id, username from users where status = 1;
SELECT ID, USERNAME FROM USERS WHERE STATUS = 1;
```

### 2.2 缩进和换行

```sql
-- ✅ 推荐：合理缩进和换行

-- 简单查询：单行
SELECT * FROM users WHERE id = 1;

-- 复杂查询：多行，每个关键字一行
SELECT
    u.id,
    u.username,
    u.email,
    COUNT(o.id) AS order_count,
    SUM(o.total_amount) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.status = 1
    AND u.created_at >= '2024-01-01'
GROUP BY u.id
HAVING COUNT(o.id) > 0
ORDER BY total_spent DESC
LIMIT 10;

-- 子查询：额外缩进
SELECT
    u.id,
    u.username,
    (
        SELECT COUNT(*)
        FROM orders
        WHERE user_id = u.id
    ) AS order_count
FROM users u;

-- CASE WHEN：每个WHEN一行
SELECT
    id,
    username,
    CASE
        WHEN age < 18 THEN '未成年'
        WHEN age < 60 THEN '成年'
        ELSE '老年'
    END AS age_group
FROM users;

-- IN列表：适当换行
SELECT * FROM users
WHERE id IN (
    1, 2, 3, 4, 5,
    6, 7, 8, 9, 10
);

-- ❌ 不推荐：
SELECT u.id, u.username, u.email, COUNT(o.id) AS order_count FROM users u LEFT JOIN orders o ON u.id = o.user_id WHERE u.status = 1 GROUP BY u.id;
```

### 2.3 注释规范

```sql
-- ✅ 推荐：

-- 单行注释：使用 --
-- 查询活跃用户
SELECT * FROM users WHERE status = 1;

-- 多行注释：使用 /* */
/*
 * 查询用户订单统计
 * 包含：订单数量、订单总额
 * 排除：已取消的订单
 */
SELECT
    u.id,
    u.username,
    COUNT(o.id) AS order_count,
    SUM(o.total_amount) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
    AND o.status != 5  -- 排除已取消订单
GROUP BY u.id;

-- 表注释
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY COMMENT '用户ID',
    username VARCHAR(50) NOT NULL COMMENT '用户名',
    email VARCHAR(100) NOT NULL COMMENT '邮箱',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';

-- ❌ 不推荐：
SELECT * FROM users WHERE status = 1;  # 避免使用#注释（MySQL特有）
```

---

## 三、可维护性最佳实践

### 3.1 避免SELECT *

```sql
-- ❌ 不推荐：SELECT *
SELECT * FROM users WHERE id = 1;

-- 问题：
-- 1. 返回不需要的字段，浪费网络带宽
-- 2. 无法使用覆盖索引
-- 3. 字段顺序变化可能导致程序bug
-- 4. 字段新增导致程序bug

-- ✅ 推荐：明确指定字段
SELECT
    id,
    username,
    email,
    created_at
FROM users
WHERE id = 1;

-- 好处：
-- 1. 清晰的字段列表
-- 2. 可能使用覆盖索引
-- 3. 字段变化不影响程序
```

### 3.2 使用别名

```sql
-- ✅ 推荐：使用表别名

-- 简单别名
SELECT
    u.id,
    u.username,
    o.order_no,
    o.total_amount
FROM users u
JOIN orders o ON u.id = o.user_id;

-- 复杂别名
SELECT
    u.id AS user_id,
    u.username AS user_name,
    COUNT(o.id) AS order_count,
    IFNULL(SUM(o.total_amount), 0) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id;

-- ❌ 不推荐：不使用别名
SELECT
    users.id,
    users.username,
    orders.order_no,
    orders.total_amount
FROM users
JOIN orders ON users.id = orders.user_id;
```

### 3.3 显式JOIN

```sql
-- ✅ 推荐：使用显式JOIN
SELECT
    u.id,
    u.username,
    o.order_no
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE u.status = 1;

-- ❌ 不推荐：隐式JOIN
SELECT
    u.id,
    u.username,
    o.order_no
FROM users u, orders o
WHERE u.id = o.user_id
    AND u.status = 1;

-- 问题：
-- 1. 不清晰
-- 2. 容易出现笛卡尔积
-- 3. 难以维护
```

### 3.4 明确JOIN类型

```sql
-- ✅ 推荐：明确JOIN类型

-- INNER JOIN：只返回匹配的行
SELECT u.*, o.*
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN：返回左表所有行
SELECT u.*, o.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- RIGHT JOIN：避免使用，改用LEFT JOIN
-- ❌
SELECT u.*, o.*
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- ✅ 改写为LEFT JOIN
SELECT u.*, o.*
FROM orders o
LEFT JOIN users u ON o.user_id = u.id;
```

---

## 四、性能最佳实践

### 4.1 WHERE条件优化

```sql
-- ✅ 推荐：

-- 1. 避免在WHERE中使用函数
-- ❌
SELECT * FROM orders WHERE YEAR(created_at) = 2024;

-- ✅
SELECT * FROM orders
WHERE created_at >= '2024-01-01'
    AND created_at < '2025-01-01';

-- 2. 避免隐式类型转换
-- ❌ phone是VARCHAR
SELECT * FROM users WHERE phone = 13800138000;

-- ✅
SELECT * FROM users WHERE phone = '13800138000';

-- 3. 避免前导模糊查询
-- ❌
SELECT * FROM users WHERE username LIKE '%san';

-- ✅
SELECT * FROM users WHERE username LIKE 'zhang%';

-- 4. 使用IN代替OR（少量值）
-- ❌
SELECT * FROM users WHERE status = 1 OR status = 2 OR status = 3;

-- ✅
SELECT * FROM users WHERE status IN (1, 2, 3);

-- 5. 避免NOT IN，使用NOT EXISTS或LEFT JOIN
-- ❌
SELECT * FROM users
WHERE id NOT IN (SELECT user_id FROM orders);

-- ✅
SELECT u.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```

### 4.2 索引优化

```sql
-- ✅ 推荐：

-- 1. WHERE条件字段加索引
CREATE INDEX idx_status ON users(status);
SELECT * FROM users WHERE status = 1;

-- 2. ORDER BY字段加索引
CREATE INDEX idx_created_at ON users(created_at);
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;

-- 3. 联合索引遵循最左前缀
CREATE INDEX idx_status_created ON users(status, created_at);

-- ✅ 使用索引
SELECT * FROM users WHERE status = 1 ORDER BY created_at;

-- ❌ 不使用索引
SELECT * FROM users WHERE created_at >= '2024-01-01';
```

### 4.3 分页优化

```sql
-- ✅ 推荐：

-- 浅分页：直接使用LIMIT
SELECT * FROM users
ORDER BY id
LIMIT 20 OFFSET 0;

-- 深分页：使用子查询或游标
-- ❌ 慢
SELECT * FROM users
ORDER BY id
LIMIT 1000000, 20;

-- ✅ 快：延迟关联
SELECT u.*
FROM users u
JOIN (
    SELECT id
    FROM users
    ORDER BY id
    LIMIT 1000000, 20
) AS t ON u.id = t.id;

-- ✅ 更快：游标分页
SELECT * FROM users
WHERE id > :last_id
ORDER BY id
LIMIT 20;
```

---

## 五、安全最佳实践

### 5.1 防止SQL注入

```python
-- ❌ 危险：字符串拼接
username = request.get('username')
sql = f"SELECT * FROM users WHERE username = '{username}'"
db.execute(sql)

-- 攻击：username = "' OR '1'='1"
-- 拼接后：SELECT * FROM users WHERE username = '' OR '1'='1'
-- 返回所有用户！

-- ✅ 安全：使用参数化查询
username = request.get('username')
sql = "SELECT * FROM users WHERE username = %s"
db.execute(sql, (username,))

-- 或使用ORM
user = User.objects.filter(username=username).first()
```

### 5.2 权限最小化

```sql
-- ✅ 推荐：

-- 应用程序只需要SELECT、INSERT、UPDATE、DELETE权限
-- 不需要CREATE、DROP、ALTER等权限

-- 创建应用程序专用用户
CREATE USER 'app_user'@'%' IDENTIFIED BY 'password';

-- 只授予必要权限
GRANT SELECT, INSERT, UPDATE, DELETE ON ecommerce_db.* TO 'app_user'@'%';

-- 不要使用root用户
-- ❌
mysql -uroot -p
```

### 5.3 敏感数据处理

```sql
-- ✅ 推荐：

-- 1. 密码加密存储（bcrypt、argon2）
-- ❌ 不要明文存储
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(100) NOT NULL  -- ❌ 明文密码
);

-- ✅ 使用bcrypt哈希
import bcrypt

password = "user_password"
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt())

INSERT INTO users (username, password)
VALUES ('zhangsan', hashed);

-- 验证密码
stored_hash = db.query_one("SELECT password FROM users WHERE username = %s", ('zhangsan',))
if bcrypt.checkpw(password.encode(), stored_hash):
    # 密码正确
    pass

-- 2. 身份证号、手机号等脱敏
SELECT
    id,
    username,
    CONCAT(LEFT(phone, 3), '****', RIGHT(phone, 4)) AS phone_masked,
    CONCAT(LEFT(id_card, 6), '********', RIGHT(id_card, 4)) AS id_card_masked
FROM users;

-- 3. 记录操作日志
CREATE TABLE audit_logs (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    action VARCHAR(50) NOT NULL,
    table_name VARCHAR(50) NOT NULL,
    record_id BIGINT UNSIGNED NOT NULL,
    old_value TEXT,
    new_value TEXT,
    ip VARCHAR(50) NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;
```

---

## 六、代码审查清单

### 6.1 命名检查

```sql
-- ✅ 数据库、表、字段名是否使用小写+下划线？
-- ✅ 表名是否使用复数？
-- ✅ 索引命名是否规范（idx_、uniq_、ft_）？
-- ✅ 约束命名是否规范（fk_、ck_）？
-- ✅ 是否避免了保留字（order、user、group等）？
```

### 6.2 性能检查

```sql
-- ✅ 是否避免了SELECT *？
-- ✅ WHERE条件字段是否有索引？
-- ✅ 是否避免了在WHERE中使用函数？
-- ✅ 是否避免了隐式类型转换？
-- ✅ 是否避免了前导模糊查询（LIKE '%xxx'）？
-- ✅ 是否避免了NOT IN？
-- ✅ 深分页是否优化？
-- ✅ 是否使用了EXPLAIN分析？
```

### 6.3 安全检查

```sql
-- ✅ 是否使用了参数化查询？
-- ✅ 是否避免了字符串拼接SQL？
-- ✅ 密码是否加密存储？
-- ✅ 敏感数据是否脱敏？
-- ✅ 是否遵循最小权限原则？
-- ✅ 是否记录了操作日志？
```

### 6.4 可维护性检查

```sql
-- ✅ SQL是否有注释？
-- ✅ 是否使用了有意义的表别名？
-- ✅ 是否使用了显式JOIN？
-- ✅ 字段是否明确列出？
-- ✅ 复杂SQL是否拆分为多步？
-- ✅ 是否容易理解和修改？
```

---

## 七、实战示例

### 7.1 标准查询示例

```sql
-- 查询用户订单统计

-- ❌ 不规范版本
select * from users u, orders o where u.id=o.user_id and u.status=1 and o.created_at>='2024-01-01' group by u.id order by sum(o.total_amount) desc limit 10;

-- ✅ 规范版本
/*
 * 查询活跃用户的订单统计
 * 统计2024年以来的订单数量和金额
 * 按订单总额降序排列，取前10名
 */
SELECT
    u.id AS user_id,
    u.username,
    u.email,
    COUNT(o.id) AS order_count,
    IFNULL(SUM(o.total_amount), 0) / 100 AS total_spent  -- 金额从分转为元
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE u.status = 1  -- 活跃用户
    AND o.status IN (2, 3, 4)  -- 已支付、已发货、已完成
    AND o.created_at >= '2024-01-01'
    AND o.deleted_at IS NULL  -- 未删除
GROUP BY u.id, u.username, u.email
HAVING COUNT(o.id) > 0  -- 至少有1个订单
ORDER BY total_spent DESC
LIMIT 10;

-- 添加索引
CREATE INDEX idx_status_created_deleted ON orders(status, created_at, deleted_at);
CREATE INDEX idx_user_id ON orders(user_id);
```

### 7.2 标准建表示例

```sql
-- ❌ 不规范版本
create table user(
  id int primary key auto_increment,
  name varchar(50),
  email varchar(100),
  password varchar(100),
  status int
);

-- ✅ 规范版本
CREATE TABLE users (
    -- 主键
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY COMMENT '用户ID',

    -- 基本信息
    username VARCHAR(50) NOT NULL COMMENT '用户名',
    email VARCHAR(100) NOT NULL COMMENT '邮箱',
    password VARCHAR(255) NOT NULL COMMENT '密码（bcrypt哈希）',
    phone VARCHAR(20) NOT NULL DEFAULT '' COMMENT '手机号',
    avatar VARCHAR(255) NOT NULL DEFAULT '' COMMENT '头像URL',

    -- 状态字段
    status TINYINT UNSIGNED NOT NULL DEFAULT 1 COMMENT '状态：1-正常，2-禁用',
    email_verified TINYINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '邮箱是否验证：0-未验证，1-已验证',

    -- 时间戳
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted_at DATETIME DEFAULT NULL COMMENT '删除时间（软删除）',

    -- 索引
    UNIQUE KEY uniq_username (username),
    UNIQUE KEY uniq_email (email),
    INDEX idx_phone (phone),
    INDEX idx_status (status),
    INDEX idx_created_at (created_at),
    INDEX idx_deleted_at (deleted_at)

) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci
  COMMENT='用户表';
```

---

## 八、本章总结

### 核心要点

1. **命名规范**：
   - 小写字母 + 下划线
   - 有意义的名称
   - 统一的前缀/后缀

2. **编写风格**：
   - 关键字大写
   - 合理缩进
   - 适当注释

3. **可维护性**：
   - 避免SELECT *
   - 使用别名
   - 显式JOIN

4. **性能**：
   - 合理使用索引
   - 优化WHERE条件
   - 避免常见陷阱

5. **安全**：
   - 参数化查询
   - 权限最小化
   - 敏感数据加密

### 下一步

完成本章学习后，你应该能够：
- ✅ 编写规范的SQL语句
- ✅ 遵循最佳实践
- ✅ 提高代码可维护性
- ✅ 避免安全漏洞
- ✅ 通过代码审查

下一节：[24-常见陷阱与解决方案](./24-常见陷阱与解决方案.md)
