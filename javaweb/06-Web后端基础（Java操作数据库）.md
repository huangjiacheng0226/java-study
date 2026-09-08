# 06 Web 后端基础（Java 操作数据库）

## 1. JDBC

### 1.1 JDBC 是什么

JDBC 是 Java 操作关系型数据库的标准 API。数据库厂商提供驱动实现，程序通过统一接口执行 SQL。

### 1.2 查询示例

~~~java
String sql = "SELECT id, username FROM user WHERE username = ?";
try (Connection conn = dataSource.getConnection();
     PreparedStatement stmt = conn.prepareStatement(sql)) {
    stmt.setString(1, username); // 绑定参数，避免 SQL 注入

    try (ResultSet rs = stmt.executeQuery()) {
        while (rs.next()) {
            System.out.println(rs.getLong("id"));
        }
    }
}
~~~

ResultSet 的 next() 移动到下一行，getXxx 按列名或列号读取数据。try-with-resources 自动关闭资源。

executeUpdate() 用于 INSERT、UPDATE、DELETE，并返回受影响行数。多条语句应使用 Connection 的事务控制，成功 commit，失败 rollback。连接池预先创建并复用连接，减少频繁建立连接的开销；Spring Boot 项目通常自动配置 HikariCP，不要在每个请求中手动创建 DriverManager 连接。

## 2. MyBatis

### 2.1 注解 Mapper

~~~java
@Mapper
public interface DeptMapper {
    @Select("SELECT id, name, create_time, update_time FROM dept")
    List<Dept> findAll();

    @Delete("DELETE FROM dept WHERE id = #{id}")
    void deleteById(Long id);
}
~~~

### 2.2 XML 动态 SQL

~~~xml
<select id="find" resultType="com.example.Dept">
  SELECT id, name FROM dept
  <where>
    <if test="name != null and name != ''">
      name LIKE CONCAT('%', #{name}, '%')
    </if>
  </where>
  ORDER BY id DESC
</select>
~~~

井号占位符表示预编译参数，应优先使用；美元占位符是字符串替换，只能用于可信表名或列名。

### 2.3 JDBC 和 MyBatis 对比

| 项目 | JDBC | MyBatis |
|---|---|---|
| 样板代码 | 多 | 少 |
| SQL 位置 | Java 字符串 | 注解或 XML |
| 结果映射 | 手动 | 自动映射 |
| 动态 SQL | 手工拼接 | if、where、foreach |

复杂关联查询可使用 resultMap 映射字段和嵌套对象；多个参数建议使用 @Param 明确名称。可复用 SQL 片段用 sql 和 include，但动态列名必须白名单校验。

## 3. 配置和分页

~~~yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/tlias?serverTimezone=Asia/Shanghai
    username: root
    password: 123456
mybatis:
  configuration:
    map-underscore-to-camel-case: true
~~~

分页需要 total 和当前页 rows。原始实现使用 LIMIT offset, pageSize，也可以使用 PageHelper 简化。

## 4. 本章总结

- JDBC 帮助理解连接、预编译、结果集和资源关闭。
- MyBatis 减少样板代码，但 SQL 仍需开发者设计。
- 参数优先使用预编译绑定，动态 SQL 使用 where、if、foreach。
