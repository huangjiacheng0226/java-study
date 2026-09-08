# 08 后端 Web 实战（员工管理：多表与分页）

## 1. 多表关系

| 关系 | 示例 | 实现 |
|---|---|---|
| 一对多 | 部门与员工 | 员工表保存 dept_id |
| 一对一 | 员工与扩展信息 | 唯一键关联 |
| 多对多 | 员工与项目 | 中间表保存两边主键 |

项目中常使用逻辑外键，由应用层维护关系，减少数据库强约束对迁移的影响。

物理外键由数据库约束引用关系，数据一致性强，但删除和迁移限制较多；逻辑外键只保存关联 ID，由应用和业务代码保证关系，项目迁移更灵活。无论采用哪种方式，关联字段都应建立索引并在删除时明确处理策略。

一对多查询时，员工表是多的一方，通常保存 `dept_id`；一对一关系要求关联字段唯一；多对多关系需要中间表保存两边主键。

设计多表关系时先确定“谁是主表、谁是从表”，再决定关联字段和删除策略。查询列表通常以员工为主表使用 `LEFT JOIN`；编辑详情时可分别查询主表和子表，避免连接多个一对多关系造成笛卡尔积。

多对多中间表示例：

~~~sql
CREATE TABLE emp_project (
  emp_id BIGINT NOT NULL,
  project_id BIGINT NOT NULL,
  PRIMARY KEY (emp_id, project_id)
);
~~~

联合主键可以防止同一员工和项目关系被重复保存。

## 2. 多表查询

连接员工和工作经历时，一名员工可能产生多行。列表查询应只连接需要展示的表，详情查询再聚合经历，或使用 resultMap 的 collection 映射。

### 2.1 多表查询类型

| 类型 | 结果特点 | 常见用途 |
|---|---|---|
| 内连接 `INNER JOIN` | 只保留两表都匹配的记录 | 只看有部门的员工 |
| 左外连接 `LEFT JOIN` | 保留左表全部记录 | 员工没有部门时也显示 |
| 右外连接 `RIGHT JOIN` | 保留右表全部记录 | 实际项目较少使用 |
| 子查询 | 一个查询嵌套另一个查询 | 先统计再筛选 |

子查询可以返回一个值、一列、多列或一张临时表，使用 `IN`、比较运算符或 `FROM` 接收结果。

### 2.2 员工和部门

~~~sql
SELECT e.id, e.name, d.name AS dept_name
FROM emp e
LEFT JOIN dept d ON e.dept_id = d.id;
~~~

使用 `LEFT JOIN` 是为了保留尚未分配部门的员工。若使用 `INNER JOIN`，没有匹配部门的员工不会出现在结果中。

常见子查询示例：

~~~sql
-- 查询工资高于全体平均工资的员工
SELECT id, name, salary
FROM emp
WHERE salary > (SELECT AVG(salary) FROM emp);

-- 查询研发部和测试部的员工
SELECT id, name
FROM emp
WHERE dept_id IN (
  SELECT id FROM dept WHERE name IN ('研发部', '测试部')
);
~~~

### 2.3 条件分页 Mapper

~~~xml
<select id="page" resultType="com.example.Emp">
  SELECT e.*, d.name AS deptName
  FROM emp e LEFT JOIN dept d ON e.dept_id = d.id
  <where>
    <if test="name != null and name != ''">
      e.name LIKE CONCAT('%', #{name}, '%')
    </if>
    <if test="gender != null">AND e.gender = #{gender}</if>
    <if test="job != null">AND e.job = #{job}</if>
  </where>
  ORDER BY e.update_time DESC
</select>
~~~

### 2.4 接口格式

~~~text
GET /emps?page=1&pageSize=10&name=张&gender=1&job=2
~~~

~~~json
{
  "code": 1,
  "msg": "success",
  "data": {
    "total": 42,
    "rows": []
  }
}
~~~

响应中的 `total` 是符合条件的总记录数，`rows` 是当前页数据。查询总数和查询当前页必须使用相同的过滤条件，否则页码会显示错误。

查询参数通常需要做默认值和范围校验：`page` 从 1 开始，`pageSize` 至少为 1 并设置最大值；字符串条件去除首尾空格；枚举条件只接受允许的值。分页返回空页时应返回 `rows: []`，不要返回 `null`。

## 3. 分页流程

~~~mermaid
flowchart TD
    A[接收页码和条件] --> B[校验默认值]
    B --> C[查询总数]
    B --> D[计算 offset]
    D --> E[查询当前页]
    C --> F[封装 PageResult]
    E --> F --> G[返回 JSON]
~~~

分页必须处理页码、页大小、偏移量、总数和空结果。动态查询优先使用 MyBatis 的 where 和 if。

分页排序必须稳定，例如使用 update_time DESC, id DESC，避免同一时间数据在不同页之间跳动。pageSize 要设置上限，防止一次查询过多数据。

原始分页方式需要手动执行 count 查询和 `LIMIT offset, pageSize`；PageHelper 可以在执行 Mapper 查询前自动改写 SQL：

~~~java
PageHelper.startPage(page, pageSize);
List<Emp> rows = empMapper.list(name, gender, job);
PageInfo<Emp> pageInfo = new PageInfo<>(rows);
return new PageResult<>(pageInfo.getTotal(), pageInfo.getList());
~~~

分页插件只应作用于紧接着的一次查询，不能在一个线程中随意复用分页参数。接口还应限制最大 `pageSize`，避免一次返回过多数据。

分页 SQL 的 `ORDER BY` 必须稳定，常用 `update_time DESC, id DESC` 作为兜底排序。列表接口只返回页面需要的字段，详情接口再查询完整对象，有助于减少网络和数据库开销。

## 4. 本章总结

- 多表查询的关键是连接条件和对象映射。
- 分页响应通常包含 total 和 rows。
- 条件为空时 SQL 仍应合法，不能产生多余的 AND。
