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

### 2.1 Mapper

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

### 2.2 Controller

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

新增部门要校验名称非空、长度和唯一性；重复提交可能造成重复数据，应通过唯一索引、请求幂等设计或业务检查处理。新增可返回 201 或统一业务成功码，删除成功可返回 204 或统一空 data。异常时不要把数据库异常原文返回给前端。

### 2.3 请求流程

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

## 3. 日志

使用 Logback 或 SLF4J 记录关键操作：

~~~java
log.info("查询部门，name={}", name);
~~~

不要记录密码、完整令牌和数据库连接密码。

## 4. 本章总结

- 部门管理实现查询、新增、修改、删除。
- Apifox 用于接口测试，Vue 用于联调。
- Controller、Service、Mapper 分层使代码更容易维护。
