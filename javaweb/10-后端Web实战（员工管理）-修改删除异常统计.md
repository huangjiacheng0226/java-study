# 10 后端 Web 实战（员工管理：修改、删除、异常和统计）

## 1. 修改与删除

### 1.1 接口流程

~~~text
GET    /emps/{id}  -> 查询回显
PUT    /emps       -> 提交修改
DELETE /emps/{id}  -> 删除员工
~~~

修改通常分为查询回显和提交更新两个阶段。删除前要考虑工作经历等关联数据和重复请求。

## 2. 全局异常处理

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

## 3. 员工统计

### 3.1 职位统计

~~~sql
SELECT job, COUNT(*) AS total
FROM emp
GROUP BY job
ORDER BY total DESC;
~~~

### 3.2 性别统计

~~~sql
SELECT gender, COUNT(*) AS total
FROM emp
GROUP BY gender;
~~~

后端把聚合结果转换成图表数组，前端使用 ECharts 等组件可视化。

统计接口可按日期和部门增加过滤条件；大数据量时考虑索引、缓存或异步汇总，避免每次请求扫描整张员工表。

## 4. 日志、认证和请求拦截

### 4.1 Session 与 JWT

| 方案 | 状态位置 | 优点 | 注意点 |
|---|---|---|---|
| Session | 服务端 | 易撤销、成熟 | 集群需共享会话 |
| JWT | 客户端令牌 | 无状态，适合分离架构 | 撤销和泄露处理复杂 |

JWT 的 Payload 不是加密内容，不能保存密码。签名保证完整性，不保证保密。

JWT 过期时间应较短，刷新令牌要单独管理；签名密钥放在环境变量或密钥管理系统。前端存储令牌时要评估 XSS 和 CSRF 风险。

### 4.2 Filter 与 Interceptor

~~~mermaid
flowchart LR
    A[请求] --> B[Filter]
    B --> C[Interceptor]
    C --> D[Controller]
    D --> E[响应]
~~~

Filter 属于 Servlet 规范，Interceptor 属于 Spring MVC。登录接口、静态资源等白名单要明确配置。

## 5. 本章总结

- 修改由回显和提交更新组成，删除要考虑关联数据。
- 全局异常处理统一响应并隐藏内部信息。
- 统计由 SQL 聚合完成，日志和认证负责系统可维护性。
