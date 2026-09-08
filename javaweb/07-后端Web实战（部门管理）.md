# 07 后端 Web 实战（部门管理）

## 1. 项目规范

### 1.1 REST 接口

| 功能 | 方法 | 路径 |
|---|---|---|
| 查询全部 | GET | /depts |
| 查询单个 | GET | /depts/{id} |
| 新增 | POST | /depts |
| 修改 | PUT | /depts |
| 删除 | DELETE | /depts/{id} |

REST 是风格约定：URL 定位资源，HTTP 方法描述操作。

REST 接口设计原则：

- URL 使用名词表示资源，例如 `/depts`、`/emps`，不要把动作写成 `/queryDept`。
- GET 用于查询，POST 用于新增，PUT 用于修改，DELETE 用于删除。
- 同一资源的不同操作通过 HTTP 方法区分。
- 接口返回统一 JSON 结构，前端不需要为每个接口猜测格式。

GET、PUT 和 DELETE 通常具有幂等性：重复执行最终结果相同；POST 通常用于新增，重复提交可能产生多条数据，所以新增接口要结合唯一约束或幂等设计。

接口文档至少要写清请求方法、URL、路径参数、查询参数、请求体、响应体、状态码和错误场景。前后端必须约定字段命名（例如 `msg` 还是 `message`）和时间格式，不能只凭接口截图猜测。

### 1.2 开发流程

~~~mermaid
flowchart TD
    A[需求分析] --> B[定义接口]
    B --> C[后端实现]
    B --> D[前端实现]
    C --> E[Apifox 测试]
    D --> F[页面测试]
    E --> G[前后端联调]
    F --> G
~~~

## 2. 部门 CRUD

### 2.1 接口约定

| 接口 | 请求参数 | 成功结果 |
|---|---|---|
| `GET /depts` | 无或查询条件 | 部门集合 |
| `GET /depts/{id}` | 路径参数 `id` | 单个部门 |
| `POST /depts` | JSON 请求体 | 新增成功 |
| `PUT /depts` | JSON 请求体，包含 `id` | 修改成功 |
| `DELETE /depts/{id}` | 路径参数 `id` | 删除成功 |

统一响应对象可以使用 `code`、`message` 和 `data`：

~~~java
public record Result<T>(int code, String message, T data) {
    public static <T> Result<T> success(T data) {
        return new Result<>(1, "success", data);
    }

    public static Result<Void> success() {
        return new Result<>(1, "success", null);
    }
}
~~~

### 2.2 Mapper

~~~java
@Mapper
public interface DeptMapper {
    @Select("SELECT id, name, create_time, update_time FROM dept ORDER BY id DESC")
    List<Dept> findAll();

    @Insert("INSERT INTO dept(name, create_time, update_time) VALUES(#{name}, NOW(), NOW())")
    void insert(Dept dept);

    @Update("UPDATE dept SET name=#{name}, update_time=NOW() WHERE id=#{id}")
    void update(Dept dept);

    @Delete("DELETE FROM dept WHERE id=#{id}")
    void deleteById(Long id);
}
~~~

### 2.3 Service

Service 负责组织业务逻辑，Controller 只负责接收参数和返回结果：

~~~java
@Service
public class DeptServiceImpl implements DeptService {
    private final DeptMapper mapper;

    public DeptServiceImpl(DeptMapper mapper) {
        this.mapper = mapper;
    }

    @Override
    public void save(Dept dept) {
        if (dept.getName() == null || dept.getName().isBlank()) {
            throw new IllegalArgumentException("部门名称不能为空");
        }
        mapper.insert(dept);
    }
}
~~~

### 2.4 Controller

~~~java
@RestController
@RequestMapping("/depts")
public class DeptController {
    private final DeptService service;

    public DeptController(DeptService service) {
        this.service = service;
    }

    @GetMapping
    public Result<List<Dept>> list() {
        return Result.success(service.list());
    }

    @PostMapping
    public Result<Void> save(@RequestBody Dept dept) {
        service.save(dept);
        return Result.success();
    }
}
~~~

参数接收方式要和请求格式匹配：

~~~java
@GetMapping("/{id}")
public Result<Dept> find(@PathVariable Long id) {
    return Result.success(service.find(id));
}

@GetMapping("/search")
public Result<List<Dept>> search(@RequestParam(required = false) String name) {
    return Result.success(service.search(name));
}

@PostMapping
public Result<Void> save(@Valid @RequestBody Dept dept) {
    service.save(dept);
    return Result.success();
}
~~~

`@RequestParam` 接收查询参数，`@PathVariable` 接收路径变量，`@RequestBody` 把 JSON 请求体转换为 Java 对象。使用 `@Valid` 后，实体类可以用 `@NotBlank`、`@Size` 等注解声明校验规则。

新增部门要校验名称非空、长度和唯一性；重复提交可能造成重复数据，应通过唯一索引、请求幂等设计或业务检查处理。新增可返回 201 或统一业务成功码，删除成功可返回 204 或统一空 data。异常时不要把数据库异常原文返回给前端。

### 2.5 请求流程

~~~mermaid
sequenceDiagram
    participant UI as Vue
    participant C as Controller
    participant S as Service
    participant M as Mapper
    participant DB as MySQL
    UI->>C: GET /depts
    C->>S: list()
    S->>M: findAll()
    M->>DB: SELECT
    DB-->>M: 数据
    M-->>C: 部门列表
    C-->>UI: JSON
~~~

### 2.6 修改和删除的实现要点

修改时先根据 ID 查询数据是否存在，再执行更新；删除时检查 Mapper 返回的受影响行数。业务层可以把“未找到记录”转换为明确的业务异常，由全局异常处理器统一返回。

~~~java
@DeleteMapping("/{id}")
public ResponseEntity<Void> delete(@PathVariable Long id) {
    service.delete(id);
    return ResponseEntity.noContent().build();
}
~~~

Controller 不应直接拼接 SQL，也不应把数据库异常原文返回给浏览器。

修改请求常见校验流程：先校验 JSON 格式和字段范围，再查询 ID 是否存在，最后执行更新并检查受影响行数。删除请求还要确认是否存在关联员工；如果不允许删除，应返回明确的业务错误，而不是让数据库报错。

## 3. 日志

使用 Logback 或 SLF4J 记录关键操作：

~~~java
log.info("查询部门，name={}", name);
~~~

不要记录密码、完整令牌和数据库连接密码。

日志级别从低到高通常是 TRACE、DEBUG、INFO、WARN、ERROR。开发环境可以使用 DEBUG 查看 SQL 和参数，生产环境应避免输出敏感信息和过量日志。

接口测试时建议按“正常请求、缺少参数、参数格式错误、资源不存在、重复提交、数据库异常”逐项验证。Apifox 中应保存请求方法、URL、请求头、请求体和预期响应，便于前后端联调。

日志格式应包含时间、级别、线程、类名和消息。业务关键操作可以记录操作者 ID、资源 ID 和结果，但必须脱敏密码、Token、身份证号等敏感字段。

## 4. 本章总结

- 部门管理实现查询、新增、修改、删除。
- Apifox 用于接口测试，Vue 用于联调。
- Controller、Service、Mapper 分层使代码更容易维护。
