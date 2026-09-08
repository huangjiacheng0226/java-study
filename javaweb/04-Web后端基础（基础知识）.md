# 04 Web 后端基础（Spring Boot 与 HTTP）

## 1. Spring Boot Web 入门

### 1.1 启动类

~~~java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
~~~

Spring Boot 通过起步依赖和自动配置减少样板代码，内嵌 Tomcat 可以直接运行。

### 1.2 Controller

~~~java
@RestController
@RequestMapping("/depts")
public class DeptController {
    @GetMapping
    public List<Dept> list() {
        return deptService.list();
    }

    @GetMapping("/{id}")
    public Dept find(@PathVariable Long id) {
        return deptService.find(id);
    }

    @PostMapping
    public void save(@RequestBody Dept dept) {
        deptService.save(dept);
    }
}
~~~

@RestController 等价于 Controller 和 ResponseBody 的组合。

## 2. HTTP 协议

### 2.1 请求和响应组成

请求由请求行、请求头、空行和请求体组成；响应由状态行、响应头、空行和响应体组成。

### 2.2 方法与状态码

| 方法 | 语义 | 参数位置 |
|---|---|---|
| GET | 查询 | 查询字符串、路径 |
| POST | 创建 | JSON 请求体 |
| PUT | 更新 | 路径和 JSON |
| DELETE | 删除 | 路径 |

状态码：200 成功、201 创建、204 无内容、400 参数错误、401 未认证、403 无权限、404 不存在、500 服务端异常。JSON 请求应使用 Content-Type: application/json；文件上传使用 multipart/form-data。统一响应对象通常包含 code、message、data，避免每个接口格式不同。

### 2.3 参数接收

~~~java
@GetMapping("/search")
public List<Dept> search(@RequestParam(required = false) String name) {
    return deptService.search(name);
}

@GetMapping("/{id}")
public Dept find(@PathVariable Long id) {
    return deptService.find(id);
}

@PostMapping
public void add(@RequestBody Dept dept) {
    deptService.save(dept);
}
~~~

简单参数用 RequestParam，路径变量用 PathVariable，JSON 用 RequestBody。日期参数应明确格式。

对 JSON 对象使用 @Valid 和约束注解，避免把非法数据带入业务层：

~~~java
public class Dept {
    @NotBlank(message = "部门名称不能为空")
    @Size(max = 10, message = "名称不能超过 10 个字符")
    private String name;
}

@PostMapping
public Result<Void> save(@Valid @RequestBody Dept dept) {
    service.save(dept);
    return Result.success();
}
~~~

校验失败时统一处理 MethodArgumentNotValidException，返回字段级错误信息。

## 3. 分层解耦

### 3.1 三层架构

~~~text
Controller -> Service 接口 -> ServiceImpl -> Mapper
~~~

Controller 适配 HTTP，Service 组织业务，Mapper 访问数据库。

### 3.2 IOC 与 DI

IOC 表示对象交给 Spring 容器创建和管理；DI 表示容器把依赖注入对象。

~~~java
@Service
public class DeptServiceImpl implements DeptService {
    private final DeptMapper mapper;

    public DeptServiceImpl(DeptMapper mapper) {
        this.mapper = mapper;
    }
}
~~~

构造器注入依赖明确，也便于单元测试。

## 4. 本章总结

- Spring Boot 快速搭建 Web 服务。
- REST 用 URL 表示资源，用 HTTP 方法表示操作。
- 分层和 IOC/DI 用于降低耦合。
