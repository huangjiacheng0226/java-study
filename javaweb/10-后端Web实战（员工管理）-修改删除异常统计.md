# 10 后端 Web 实战（员工管理：修改、删除、异常和统计）

## 1. 修改与删除

### 1.1 接口流程

~~~text
GET    /emps/{id}  -> 查询回显
PUT    /emps       -> 提交修改
DELETE /emps/{id}  -> 删除员工
~~~

修改通常分为查询回显和提交更新两个阶段。删除前要考虑工作经历等关联数据和重复请求。

修改接口一般先通过 ID 查询详情，前端回显表单后提交完整或部分字段。更新时必须校验 ID 是否存在、字段是否合法，并只修改允许修改的字段。删除员工前要明确工作经历等子表记录是级联删除、逻辑删除还是禁止删除。

删除接口应检查受影响行数，删除不存在的 ID 时返回明确的业务结果。重复点击删除按钮不会产生额外副作用，但前端仍应在请求发送后禁用按钮，避免重复请求。

更新操作应使用白名单字段，避免客户端提交 `createTime`、`password` 或权限字段导致越权修改。数据库层可使用 `WHERE id = ?`，Service 层检查返回行数为 1；若为 0，说明记录不存在或已被其他请求修改。

修改接口参数可以使用路径变量和 JSON 请求体：

~~~java
@PutMapping("/{id}")
public Result<Void> update(@PathVariable Long id,
                           @Valid @RequestBody EmpUpdateRequest request) {
    service.update(id, request);
    return Result.success();
}
~~~

更新 DTO 只暴露允许修改的字段，不要直接把数据库实体作为所有接口的输入对象。

## 2. 全局异常处理

局部 `try/catch` 会让每个 Controller 重复处理错误。全局异常处理器可以集中记录日志、转换响应和隐藏内部实现细节。

~~~java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public Result<Void> handle(Exception ex) {
        log.error("系统异常", ex);
        return Result.error("操作失败，请稍后重试");
    }
}
~~~

详细堆栈写入日志，响应只返回稳定错误码和用户可理解的消息，不要暴露 SQL、路径和密码。

参数校验异常返回 400，认证失败返回 401，权限不足返回 403，资源不存在返回 404，未预期异常记录日志并返回 500。不要对外暴露堆栈。

建议按异常类型处理：

| 异常类型 | HTTP 状态码 | 前端可见信息 |
|---|---:|---|
| 参数校验异常 | 400 | 哪个字段不合法 |
| 未登录异常 | 401 | 请先登录 |
| 权限异常 | 403 | 没有访问权限 |
| 数据不存在 | 404 | 资源不存在 |
| 业务冲突 | 409 | 例如名称已存在 |
| 未预期异常 | 500 | 操作失败，请稍后重试 |

日志中可以保存异常堆栈、请求路径和 traceId，但响应中不要返回 SQL、文件路径、堆栈和密码。

## 3. 员工统计

### 3.1 职位统计

~~~sql
SELECT job, COUNT(*) AS total
FROM emp
GROUP BY job
ORDER BY total DESC;
~~~

`GROUP BY` 按职位分组，`COUNT(*)` 统计每组记录数量，`ORDER BY total DESC` 按统计结果从多到少排序。统计接口返回的数据通常转换为图表需要的 `name` 和 `value` 数组。

### 3.2 性别统计

~~~sql
SELECT gender, COUNT(*) AS total
FROM emp
GROUP BY gender;
~~~

后端把聚合结果转换成图表数组，前端使用 ECharts 等组件可视化。

统计接口可按日期和部门增加过滤条件；大数据量时考虑索引、缓存或异步汇总，避免每次请求扫描整张员工表。

图表接口常返回简单的统计 DTO：

~~~java
public record StatItem(String name, long value) {
}
~~~

SQL 的统计字段应使用明确别名，空结果也返回空数组而不是 `null`，这样前端可以直接遍历渲染。

## 4. 日志、认证和请求拦截

### 4.1 Session 与 JWT

| 方案 | 状态位置 | 优点 | 注意点 |
|---|---|---|---|
| Session | 服务端 | 易撤销、成熟 | 集群需共享会话 |
| JWT | 客户端令牌 | 无状态，适合分离架构 | 撤销和泄露处理复杂 |

JWT 的 Payload 不是加密内容，不能保存密码。签名保证完整性，不保证保密。

JWT 过期时间应较短，刷新令牌要单独管理；签名密钥放在环境变量或密钥管理系统。前端存储令牌时要评估 XSS 和 CSRF 风险。

Session 把会话数据放在服务端，浏览器只保存 Session ID；JWT 把声明和签名放在令牌中，服务端可以无状态校验。JWT 的 Payload 只是 Base64Url 编码，不是加密内容，不能保存密码、银行卡号等敏感信息。

### 4.2 Filter 与 Interceptor

~~~mermaid
flowchart LR
    A[请求] --> B[Filter]
    B --> C[Interceptor]
    C --> D[Controller]
    D --> E[响应]
~~~

Filter 属于 Servlet 规范，Interceptor 属于 Spring MVC。登录接口、静态资源等白名单要明确配置。

Filter 可以在进入 Spring MVC 前处理所有 Servlet 请求，适合字符编码、跨域和登录令牌解析；Interceptor 可以在 Controller 执行前后处理，适合登录校验、权限判断和操作日志。请求链路通常是 Filter → Interceptor → Controller → Service。

认证拦截的一般流程：

~~~mermaid
flowchart TD
    A[收到请求] --> B{是否白名单}
    B -->|是| C[继续处理]
    B -->|否| D[读取 Cookie 或 Authorization]
    D --> E{令牌有效}
    E -->|否| F[返回 401]
    E -->|是| G[写入登录用户信息]
    G --> C
~~~

白名单应包含登录、静态资源和健康检查接口，其他接口默认需要认证。权限校验不能只依赖前端按钮隐藏，后端必须再次验证。

Filter 适用于所有 Servlet 请求，Interceptor 只拦截进入 Spring MVC 的请求；静态资源、错误页和文件请求是否经过拦截要结合配置验证。认证成功后可把用户 ID 放入请求上下文，但线程池异步任务结束时要清理 ThreadLocal，避免用户信息串线。

## 5. 本章总结

- 修改由回显和提交更新组成，删除要考虑关联数据。
- 全局异常处理统一响应并隐藏内部信息。
- 统计由 SQL 聚合完成，日志和认证负责系统可维护性。
