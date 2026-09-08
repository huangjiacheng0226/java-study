# 06 Web 后端基础（Java 操作数据库）

## 1. JDBC

### 1.1 JDBC 是什么

JDBC 是 Java 操作关系型数据库的标准 API。数据库厂商提供驱动实现，程序通过统一接口执行 SQL。

JDBC 的基本对象包括：`Driver` 驱动、`Connection` 连接、`Statement/PreparedStatement` 语句、`ResultSet` 结果集。标准流程是加载驱动、获取连接、预编译 SQL、绑定参数、执行、读取结果、关闭资源。

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

连接对象默认可能处于自动提交模式。需要把多条 SQL 作为一个整体时，应关闭自动提交，并在成功后提交、失败后回滚：

~~~java
try (Connection conn = dataSource.getConnection()) {
    conn.setAutoCommit(false);
    try {
        // 执行多条相关 SQL
        conn.commit();
    } catch (Exception ex) {
        conn.rollback();
        throw ex;
    }
}
~~~

executeUpdate() 用于 INSERT、UPDATE、DELETE，并返回受影响行数。多条语句应使用 Connection 的事务控制，成功 commit，失败 rollback。连接池预先创建并复用连接，减少频繁建立连接的开销；Spring Boot 项目通常自动配置 HikariCP，不要在每个请求中手动创建 DriverManager 连接。

增删改示例：

~~~java
String sql = "UPDATE dept SET name = ? WHERE id = ?";
try (Connection conn = dataSource.getConnection();
     PreparedStatement stmt = conn.prepareStatement(sql)) {
    stmt.setString(1, "技术部");
    stmt.setLong(2, 1L);
    int affected = stmt.executeUpdate();
    if (affected != 1) {
        throw new IllegalStateException("部门不存在");
    }
}
~~~

不要使用字符串拼接用户输入，例如 `"... WHERE name='" + name + "'"`，应使用 `?` 参数占位符防止 SQL 注入。

JDBC 中 `Statement` 直接执行字符串 SQL，容易产生注入；`PreparedStatement` 使用占位符并预编译，适合带参数的 SQL；`CallableStatement` 用于调用存储过程。生产代码一般优先使用 `PreparedStatement`。

## 2. MyBatis

MyBatis 是持久层框架，它负责参数绑定、SQL 执行和结果映射，但 SQL 仍由开发者编写。它适合需要精确控制 SQL、动态条件和复杂关联查询的项目。

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

`@Mapper` 标记 Mapper 接口，MyBatis 会为接口创建代理对象。`#{id}` 会生成预编译参数，推荐用于用户输入；返回集合时要确认实体字段和数据库列能正确映射。

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

动态 SQL 常用标签：`<if>` 按条件拼接，`<where>` 自动处理多余的 AND/WHERE，`<set>` 处理更新语句的逗号，`<foreach>` 生成 IN 条件或批量语句。

~~~xml
<select id="findByIds" resultType="com.example.Dept">
  SELECT id, name FROM dept
  <where>
    <if test="ids != null and ids.size() > 0">
      id IN
      <foreach collection="ids" item="id" open="(" separator="," close=")">
        #{id}
      </foreach>
    </if>
  </where>
</select>
~~~

### 2.3 JDBC 和 MyBatis 对比

| 项目 | JDBC | MyBatis |
|---|---|---|
| 样板代码 | 多 | 少 |
| SQL 位置 | Java 字符串 | 注解或 XML |
| 结果映射 | 手动 | 自动映射 |
| 动态 SQL | 手工拼接 | if、where、foreach |

复杂关联查询可使用 resultMap 映射字段和嵌套对象；多个参数建议使用 @Param 明确名称。可复用 SQL 片段用 sql 和 include，但动态列名必须白名单校验。

多参数方法建议显式命名：

~~~java
List<Dept> find(@Param("name") String name, @Param("status") Integer status);
~~~

`resultMap` 适合数据库列名和 Java 属性名不同，或需要映射嵌套对象、集合的情况。`<sql>` 和 `<include>` 可以复用列清单，但不要把未经校验的列名直接放入 `${}`。

增删改查标签的返回值要和业务语义一致：`<select>` 返回对象或集合，`<insert>`、`<update>`、`<delete>` 通常返回受影响行数。插入主键回填可以使用 `useGeneratedKeys="true" keyProperty="id"`，之后再保存依赖该主键的子表记录。

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

分页计算公式：`offset = (page - 1) * pageSize`。接口通常先查询总数，再查询当前页数据，最后封装为：

~~~java
public record PageResult<T>(long total, List<T> rows) {
}
~~~

MyBatis 常用配置还包括驼峰命名映射、日志实现和 Mapper 扫描：

~~~yaml
mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.example.pojo
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
~~~

### 3.1 XML Mapper 基本结构

XML 文件的 namespace 应与 Mapper 接口全限定名一致，标签的 `id` 应与接口方法名一致：

~~~xml
<mapper namespace="com.example.mapper.DeptMapper">
  <select id="findById" resultType="com.example.pojo.Dept">
    SELECT id, name, create_time, update_time
    FROM dept
    WHERE id = #{id}
  </select>
</mapper>
~~~

`resultType` 适合简单自动映射，`resultMap` 适合字段名不同或嵌套对象。数据库下划线字段和 Java 驼峰属性可以通过 `map-underscore-to-camel-case` 自动转换。

### 3.2 连接池和资源管理

连接池负责提前创建并复用数据库连接。请求结束后调用 `close()` 通常是把连接归还连接池，而不是物理销毁。连接不关闭会导致连接池耗尽，表现为请求越来越慢甚至无法访问数据库。

### 3.3 Lombok 常用注解

`@Data` 生成 getter、setter、`toString` 和 `equals`，`@NoArgsConstructor` 生成无参构造器，`@AllArgsConstructor` 生成全参构造器。实体类使用 Lombok 时仍要注意无参构造器、序列化和字段命名要求。

配置文件中的敏感信息应通过环境变量或配置中心注入；不要把数据库密码、云存储密钥写入公开仓库。常见配置优先级由命令行参数、环境变量、配置文件等共同决定，排错时要确认最终生效值。

## 4. 本章总结

- JDBC 帮助理解连接、预编译、结果集和资源关闭。
- MyBatis 减少样板代码，但 SQL 仍需开发者设计。
- 参数优先使用预编译绑定，动态 SQL 使用 where、if、foreach。
