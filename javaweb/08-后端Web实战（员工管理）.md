# 08 后端 Web 实战（员工管理：多表与分页）

## 1. 多表关系

| 关系 | 示例 | 实现 |
|---|---|---|
| 一对多 | 部门与员工 | 员工表保存 dept_id |
| 一对一 | 员工与扩展信息 | 唯一键关联 |
| 多对多 | 员工与项目 | 中间表保存两边主键 |

项目中常使用逻辑外键，由应用层维护关系，减少数据库强约束对迁移的影响。

## 2. 多表查询

连接员工和工作经历时，一名员工可能产生多行。列表查询应只连接需要展示的表，详情查询再聚合经历，或使用 resultMap 的 collection 映射。

### 2.1 员工和部门

~~~sql
SELECT e.id, e.name, d.name AS dept_name
FROM emp e
LEFT JOIN dept d ON e.dept_id = d.id;
~~~

### 2.2 条件分页 Mapper

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

### 2.3 接口格式

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

## 4. 本章总结

- 多表查询的关键是连接条件和对象映射。
- 分页响应通常包含 total 和 rows。
- 条件为空时 SQL 仍应合法，不能产生多余的 AND。
