# 05 Web 后端基础（MySQL 数据库）

## 1. MySQL 与关系型数据库

### 1.1 基本概念

关系型数据库使用表、行、列保存结构化数据。主键唯一标识记录，约束用于保证数据质量。

数据库是存放和管理数据的软件，MySQL 是一种关系型数据库管理系统。关系型数据库使用二维表表达数据，表之间通过主键和外键字段建立联系。

常见概念：

| 概念 | 含义 |
|---|---|
| 数据库 | 多张相关表的集合 |
| 表 | 保存同一类数据的结构 |
| 行 | 一条完整记录 |
| 列 | 记录中的一个字段 |
| 主键 | 唯一标识一行，不能重复且不能为 NULL |
| 外键 | 保存另一张表主键，用于表达关联 |
| 约束 | 限制数据取值，保证数据质量 |

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

常用字段类型包括整数 `INT`、小数 `DECIMAL`、字符串 `VARCHAR`、日期时间 `DATETIME` 和大文本 `TEXT`。表设计时应根据业务含义选择类型，不要所有字段都使用字符串。

### 1.3 数据库设计步骤

1. 根据需求识别实体，例如部门、员工和工作经历。
2. 为每个实体设计表和字段。
3. 为每张表选择主键，通常使用自增整数或业务无关的唯一 ID。
4. 分析一对一、一对多和多对多关系。
5. 为查询、关联和唯一性要求设计约束和索引。
6. 使用测试数据验证新增、修改、删除和查询。

字段名应表达清晰含义，时间字段通常使用 `create_time`、`update_time`，状态字段使用数字或枚举并在文档中说明每个值代表什么。

连接 MySQL 后常用命令：

~~~text
mysql -u root -p
SHOW DATABASES;
USE tlias;
SELECT DATABASE();
EXIT;
~~~

图形化工具（如 IDEA Database、DataGrip、Navicat）本质上也是通过驱动连接数据库。连接失败时检查主机、端口、用户名、密码、数据库名和账号权限；不要把生产密码提交到 Git。

## 2. SQL 分类

### 2.1 DDL

DDL 管理数据库和表结构，如 CREATE、ALTER、DROP。

~~~sql
ALTER TABLE dept ADD COLUMN status TINYINT NOT NULL DEFAULT 1;
ALTER TABLE dept MODIFY COLUMN name VARCHAR(20) NOT NULL;
ALTER TABLE dept DROP COLUMN status;
~~~

生产环境执行 `DROP`、`TRUNCATE` 和结构变更前必须确认目标范围并备份数据。

查看结构和表信息：

~~~sql
SHOW DATABASES;
SHOW TABLES;
DESC dept;
SHOW CREATE TABLE dept;
~~~

删除和清空的区别：`DROP TABLE` 删除表结构和数据，`TRUNCATE TABLE` 快速清空数据并保留表结构，`DELETE` 按条件删除行并可回滚（是否可回滚还取决于存储引擎和事务）。

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

`INSERT` 增加记录，`UPDATE` 修改已有记录，`DELETE` 删除记录。批量插入可以一次写入多组值，修改和删除没有 `WHERE` 时可能影响整张表。

~~~sql
INSERT INTO dept(name) VALUES ('研发部'), ('测试部');

UPDATE dept
SET name = '技术研发部'
WHERE id = 1;

DELETE FROM dept
WHERE id = 2;
~~~

### 2.3 DQL

~~~sql
SELECT id, name
FROM dept
WHERE name LIKE CONCAT('%', '研', '%')
ORDER BY id DESC
LIMIT 0, 10;
~~~

SQL 常见逻辑顺序：FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT。

常见条件运算符：`=`、`<>`、`>`、`<`、`BETWEEN`、`IN`、`LIKE`、`IS NULL`。`LIKE '%关键字%'` 可以模糊匹配，但前置通配符通常不容易使用普通索引。

常见聚合函数：`COUNT` 统计数量，`SUM` 求和，`AVG` 求平均，`MAX` 求最大值，`MIN` 求最小值。`WHERE` 在分组前过滤行，`HAVING` 在分组后过滤组。

基础查询示例：

~~~sql
SELECT id, name FROM dept;
SELECT DISTINCT job FROM emp;
SELECT * FROM emp WHERE salary BETWEEN 5000 AND 10000;
SELECT * FROM emp WHERE job IN (1, 2, 3);
SELECT * FROM emp WHERE name LIKE '张%';
SELECT * FROM emp ORDER BY update_time DESC, id DESC LIMIT 0, 10;
~~~

NULL 不是 0 或空字符串。判断空值要用 IS NULL/IS NOT NULL，不能写 = NULL；WHERE 条件可能产生 TRUE、FALSE、UNKNOWN 三种结果。主键应稳定且不重复，UNIQUE 防止业务字段重复，NOT NULL 保证必填；设计表时减少重复数据，报表场景可在可控范围内反规范化。

分页常见写法是 `LIMIT offset, pageSize`，例如第 2 页、每页 10 条为 `LIMIT 10, 10`。必须配合稳定的 `ORDER BY`，否则数据新增或更新时页内顺序可能变化。

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

连接条件必须写在 `ON` 后。`INNER JOIN` 只返回两边匹配的记录；`LEFT JOIN` 保留左表全部记录，右表没有匹配时字段为 NULL。多表查询时应明确列名，避免使用 `SELECT *` 返回不需要的字段。

子查询按返回结果可以分为标量子查询、列子查询、行子查询和表子查询：

~~~sql
-- 标量子查询：只返回一个值
SELECT * FROM emp WHERE salary > (SELECT AVG(salary) FROM emp);

-- 列子查询：配合 IN
SELECT * FROM emp WHERE dept_id IN (SELECT id FROM dept WHERE name LIKE '%技术%');

-- 表子查询：把查询结果当作临时表
SELECT t.job, t.total
FROM (SELECT job, COUNT(*) AS total FROM emp GROUP BY job) t;
~~~

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

事务通常由 `START TRANSACTION` 开始，以 `COMMIT` 提交，以 `ROLLBACK` 回滚。事务的四个特性是：原子性、一致性、隔离性和持久性。转账等必须全部成功或全部失败的操作应放在同一个事务中。

### 4.2 索引

索引适合经常用于 WHERE、JOIN、ORDER BY 的列，但会占空间并增加写入成本。

联合索引要考虑最左匹配原则；不要对索引列做函数或隐式类型转换；使用 EXPLAIN 检查查询计划。

索引本质上是帮助数据库快速定位记录的数据结构。主键和唯一约束通常会自动创建索引，但索引并非越多越好，因为插入、更新和删除也需要维护索引。

~~~sql
CREATE INDEX idx_emp_dept ON emp(dept_id);
EXPLAIN SELECT * FROM emp WHERE dept_id = 1;
~~~

联合索引 `(dept_id, update_time)` 可以优先匹配最左边的 `dept_id`；查询条件对索引列进行函数运算或隐式类型转换，可能导致索引失效。

### 4.3 事务隔离和并发问题

多个事务同时操作数据时，可能出现脏读、不可重复读和幻读。数据库通过隔离级别控制一个事务可以看到哪些并发修改。初学阶段应记住：需要保持业务整体性的多条 SQL 必须放在同一事务中，提交前的修改在失败时要回滚。

### 4.4 规范化和反规范化

规范化通过拆分表、减少重复字段来避免更新异常；反规范化是在读取性能或报表场景下有意识地保存冗余字段。反规范化必须定义同步策略，否则容易产生不一致数据。

## 5. 本章总结

- DDL 管结构，DML 管数据，DQL 查数据。
- 掌握条件、排序、分页、聚合、连接和子查询。
- 事务保证整体性，索引需要权衡读写成本。
