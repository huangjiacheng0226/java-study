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

报请求文 = 请求行 + 请求头 + 空行 + 请求体
~~~text
POST /api/user/login HTTP/1.1           ← 请求行
Host: www.example.com                   ← 请求头
Content-Type: application/json
Content-Length: 56
                                        ← 空行（必须）
{"username":"admin","password":"123456"} ← 请求体（POST 独有）
~~~

响应报文 = 响应行 + 响应头 + 空行 + 响应体
~~~text
HTTP/1.1 200 OK                         ← 响应行
Content-Type: application/json          ← 响应头
Content-Length: 85
                                        ← 空行（必须）
{"code":200,"msg":"登录成功","token":"xxx"} ← 响应体
~~~

### 2.2
- 基于TCP协议: 面向连接，安全
TCP是一种面向连接的(建立连接之前是需要经过三次握手)、可靠的、基于字节流的传输层通信协议，在数据传输方面更安全

- 基于请求-响应模型:   一次请求对应一次响应（先请求后响应）
请求和响应是一一对应关系，没有请求，就没有响应

- HTTP协议是无状态协议:  对于数据没有记忆能力。每次请求-响应都是独立的
无状态指的是客户端发送HTTP请求给服务端之后，服务端根据请求响应数据，响应完后，不会记录任何信息。
  - 缺点:  多次请求间不能共享数据
  - 优点:  速度快

- 请求之间无法共享数据会引发的问题：
  - 如：京东购物。加入购物车和去购物车结算是两次请求
  - 由于HTTP协议的无状态特性，加入购物车请求响应结束后，并未记录加入购物车是何商品
  - 发起去购物车结算的请求后，因为无法获取哪些商品加入了购物车，会导致此次请求无法正确展示数据

- 具体使用的时候，我们发现京东是可以正常展示数据的，原因是Java早已考虑到这个问题，并提出了使用会话技术(Cookie、Session)来解决这个问题。

### 2.3 方法与状态码

| 方法 | 语义 | 参数位置 |
|---|---|---|
| GET | 查询 | 查询字符串、路径 |
| POST | 创建 | JSON 请求体 |
| PUT | 更新 | 路径和 JSON |
| DELETE | 删除 | 路径 |

状态码：200 成功、201 创建、204 无内容、400 参数错误、401 未认证、403 无权限、404 不存在、500 服务端异常。JSON 请求应使用 Content-Type: application/json；文件上传使用 multipart/form-data。统一响应对象通常包含 code、message、data，避免每个接口格式不同。

### 2.4 参数接收

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
