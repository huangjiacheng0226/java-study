# 05 Web 后端基础（MySQL 数据库）

## 1. MySQL 与关系型数据库

### 1.1 基本概念

关系型数据库使用表、行、列保存结构化数据。主键唯一标识记录，约束用于保证数据质量。

### 1.2 建库建表

~~~sql
CREATE DATABASE IF NOT EXISTS tlias DEFAULT CHARACTER SET utf8mb4;
USE tlias;

CREATE TABLE dept (
  id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(10) NOT NULL UNIQUE,
  create_time DATETIME,
  update_time DATETIME
);
~~~

## 2. SQL 分类

### 2.1 DDL

DDL 管理数据库和表结构，如 CREATE、ALTER、DROP。

### 2.2 DML

~~~sql
INSERT INTO dept(name, create_time, update_time)
VALUES ('教研部', NOW(), NOW());

UPDATE dept
SET name = '技术部', update_time = NOW()
WHERE id = 1;

DELETE FROM dept WHERE id = 1;
~~~

更新和删除必须写 WHERE。执行生产操作前，先用相同条件查询确认范围。

### 2.3 DQL

~~~sql
SELECT id, name
FROM dept
WHERE name LIKE CONCAT('%', '研', '%')
ORDER BY id DESC
LIMIT 0, 10;
~~~

SQL 常见逻辑顺序：FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT。

NULL 不是 0 或空字符串。判断空值要用 IS NULL/IS NOT NULL，不能写 = NULL；WHERE 条件可能产生 TRUE、FALSE、UNKNOWN 三种结果。主键应稳定且不重复，UNIQUE 防止业务字段重复，NOT NULL 保证必填；设计表时减少重复数据，报表场景可在可控范围内反规范化。

## 3. 聚合和多表查询

~~~sql
SELECT job, COUNT(*) AS total
FROM emp
GROUP BY job
HAVING COUNT(*) > 1
ORDER BY total DESC;

SELECT e.id, e.name, d.name AS dept_name
FROM emp e
LEFT JOIN dept d ON e.dept_id = d.id;
~~~

| 类型 | 特点 | 用途 |
|---|---|---|
| INNER JOIN | 只保留匹配行 | 查询有关联的数据 |
| LEFT JOIN | 保留左表所有行 | 主表记录不能丢失 |
| 子查询 | 查询结果参与外层 | 表达复杂条件 |

## 4. 事务和索引

### 4.1 事务

事务具有原子性、一致性、隔离性和持久性。

~~~sql
START TRANSACTION;
UPDATE account SET balance = balance - 100 WHERE id = 1;
UPDATE account SET balance = balance + 100 WHERE id = 2;
COMMIT;
-- 异常时执行 ROLLBACK
~~~

### 4.2 索引

索引适合经常用于 WHERE、JOIN、ORDER BY 的列，但会占空间并增加写入成本。

联合索引要考虑最左匹配原则；不要对索引列做函数或隐式类型转换；使用 EXPLAIN 检查查询计划。

## 5. 本章总结

- DDL 管结构，DML 管数据，DQL 查数据。
- 掌握条件、排序、分页、聚合、连接和子查询。
- 事务保证整体性，索引需要权衡读写成本。
