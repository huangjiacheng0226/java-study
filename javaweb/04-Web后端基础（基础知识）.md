# 04 Web 后端基础（Spring Boot、HTTP、分层与 IoC/DI）

本篇对应课程中的 Spring Boot Web 入门、HTTP 协议、请求与响应、三层架构、分层解耦和 Spring IoC/DI。学习顺序建议为：先理解浏览器和服务器如何通信，再学习 Spring Boot 如何接收请求，最后理解对象如何交给容器管理。

## 1. Web 后端与 Spring Boot

### 1.1 静态资源和动态资源

静态资源是服务器中内容基本不随请求改变的文件，例如 HTML、CSS、JavaScript、图片、音频和视频。

动态资源是服务器根据请求、用户信息和数据库数据动态生成的内容，例如用户列表、订单详情和登录结果。早期 Java Web 常使用 Servlet、JSP，现在企业项目通常使用 Spring 体系完成动态请求处理。

### 1.2 BS 架构和 CS 架构

| 架构 | 含义 | 优点 | 缺点 |
|---|---|---|---|
| BS | Browser/Server，浏览器/服务器 | 客户端只需要浏览器，维护方便 | 对网络依赖较强，复杂交互体验受限 |
| CS | Client/Server，客户端/服务器 | 交互体验较好，客户端能力强 | 需要单独安装和维护客户端 |

Java Web 项目通常采用 BS 架构：浏览器发送 HTTP 请求，服务器处理业务并返回 HTTP 响应。

### 1.3 Spring、Spring Framework 与 Spring Boot

Spring 是一个完整的开发生态，Spring Framework 是其中的基础框架，提供 IoC、依赖注入、事务管理、Web 支持和数据访问等能力。

直接使用 Spring Framework 时需要编写较多配置。Spring Boot 在 Spring 基础上提供：

- 自动配置：根据依赖和配置自动创建常用对象。
- 起步依赖：用一个依赖引入某个场景所需的一组库。
- 内嵌服务器：项目可以直接启动内嵌 Tomcat。
- 统一配置：使用 `application.properties` 或 `application.yml` 管理参数。

### 1.4 Spring Boot 工程和启动类

可以使用 Spring Initializr 创建项目，选择 Spring Web、Lombok 等依赖。工程通常包含以下目录：

~~~text
src/main/java       Java 源代码
src/main/resources  配置文件和静态资源
src/test/java       测试代码
pom.xml             Maven 依赖和构建配置
~~~

启动类通常使用 `@SpringBootApplication`：

~~~java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
~~~

执行 `main` 方法后，Spring Boot 会创建 Spring 容器并启动内嵌 Tomcat，默认监听 `8080` 端口。可以通过配置修改端口：

~~~properties
server.port=8081
~~~

### 1.5 起步依赖和内嵌 Tomcat

`spring-boot-starter-web` 是 Web 开发的起步依赖，它会传递引入 Spring MVC、JSON 处理和 Tomcat 等依赖。Maven 的传递依赖机制让项目不需要逐个手动添加底层库。

启动流程可以概括为：

~~~mermaid
flowchart TD
    A[执行 main 方法] --> B[创建 SpringApplication]
    B --> C[加载自动配置]
    C --> D[创建 IoC 容器]
    D --> E[扫描并创建 Bean]
    E --> F[启动内嵌 Tomcat]
    F --> G[等待浏览器请求]
~~~

## 2. Controller 入门

### 2.1 第一个请求处理方法

`@RestController` 表示当前类是请求处理类，并且方法返回值直接写入响应体。`@RequestMapping` 用于设置请求路径。

~~~java
package com.example.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String hello(String name) {
        return "Hello " + name;
    }
}
~~~

浏览器访问 `http://localhost:8080/hello?name=Tom`，Spring 会把查询参数 `name` 绑定到方法参数，响应内容为 `Hello Tom`。

### 2.2 常用请求映射注解

| 注解 | 常用方法 | 示例 |
|---|---|---|
| `@RequestMapping` | 任意或指定方法 | `@RequestMapping("/users")` |
| `@GetMapping` | GET | `@GetMapping("/{id}")` |
| `@PostMapping` | POST | `@PostMapping` |
| `@PutMapping` | PUT | `@PutMapping("/{id}")` |
| `@DeleteMapping` | DELETE | `@DeleteMapping("/{id}")` |

类上的 `@RequestMapping` 可以提供公共路径，方法上的映射再补充具体路径。

### 2.3 Spring Boot 参数绑定

| 参数类型 | 示例写法 | 说明 |
|---|---|---|
| 简单参数 | `@RequestParam String name` | 接收查询参数或表单参数 |
| 路径参数 | `@PathVariable Long id` | 接收 `/users/{id}` 中的值 |
| JSON 对象 | `@RequestBody User user` | 读取 `application/json` 请求体 |
| 文件参数 | `@RequestParam MultipartFile file` | 读取 multipart 文件 |
| 日期参数 | `@DateTimeFormat(pattern = "yyyy-MM-dd")` | 把字符串转换为日期 |

~~~java
@GetMapping("/users/{id}")
public Result<User> find(@PathVariable Long id) {
    return Result.success(userService.find(id));
}

@PostMapping("/users")
public Result<Void> save(@Valid @RequestBody User user) {
    userService.save(user);
    return Result.success();
}
~~~

参数名不一致时显式指定名称，例如 `@RequestParam("user_name") String userName`。请求体 JSON 的字段名应与 Java 属性匹配，或通过 `@JsonProperty` 指定映射。

## 3. HTTP 协议

### 3.1 HTTP 的定义

HTTP 是 Hyper Text Transfer Protocol，即超文本传输协议。它规定浏览器和服务器传输数据时使用的格式。

HTTP 具有以下特点：

1. 基于 TCP，通信前建立连接，数据传输可靠。
2. 基于请求-响应模型，一次请求通常对应一次响应。
3. 无状态，服务器默认不会记住前一次请求。Cookie、Session 和 Token 可以用来保存会话信息。

### 3.2 HTTP 请求的整体结构

浏览器发送给服务器的数据称为请求协议，按顺序包括：

1. 请求行
2. 请求头
3. 空行
4. 请求体

~~~text
请求行
请求头 1
请求头 2

请求体
~~~

空行用于标记请求头结束。GET 请求的参数通常放在 URL 查询字符串中，因此一般没有请求体；POST、PUT 等请求通常把参数放在请求体中。

### 3.3 请求行的定义和格式

请求行是 HTTP 请求的第一行，由请求方法、资源路径和协议版本组成，三者使用空格分隔：

~~~http
GET /users?page=1 HTTP/1.1
~~~

| 部分 | 含义 | 示例 |
|---|---|---|
| 请求方法 | 希望服务器执行的操作 | `GET`、`POST`、`PUT`、`DELETE` |
| 资源路径 | 要访问的资源地址 | `/users` |
| 查询参数 | `?` 后的键值对 | `page=1&size=10` |
| 协议版本 | 使用的 HTTP 版本 | `HTTP/1.1` |

查询参数使用 `key=value` 表示，多个参数用 `&` 连接，路径和查询参数之间用 `?` 连接。

### 3.4 请求头的定义和常见字段

请求头从第二行开始，每行使用 `key: value` 格式。请求头描述客户端能力、希望接收的数据格式、请求体类型和客户端信息。

| 请求头 | 含义 |
|---|---|
| `Host` | 请求的主机和端口 |
| `User-Agent` | 浏览器或客户端类型 |
| `Accept` | 客户端能够接收的响应类型 |
| `Accept-Language` | 客户端偏好的语言 |
| `Accept-Encoding` | 客户端支持的压缩方式 |
| `Content-Type` | 请求体的数据类型 |
| `Content-Length` | 请求体大小，单位为字节 |
| `Cookie` | 客户端携带的会话数据 |

例如 JSON 请求通常包含：

~~~http
Content-Type: application/json
Accept: application/json
~~~

### 3.5 请求体的定义和格式

请求体位于请求头后的空行之后，用于保存提交给服务器的数据。请求体格式由 `Content-Type` 决定。

~~~http
POST /users HTTP/1.1
Content-Type: application/json

{"name":"Tom","age":18}
~~~

常见请求体类型：

| Content-Type | 典型场景 |
|---|---|
| `application/x-www-form-urlencoded` | 普通表单键值对 |
| `application/json` | 前后端分离项目提交 JSON |
| `multipart/form-data` | 文件上传和表单混合提交 |

### 3.6 GET 和 POST 对比

| 对比项 | GET | POST |
|---|---|---|
| 参数位置 | 请求行的查询字符串 | 请求体 |
| 典型用途 | 查询资源 | 创建或提交数据 |
| 是否适合缓存 | 通常可以缓存 | 通常不直接缓存 |
| URL 是否暴露参数 | 会 | 不会放在 URL 中 |
| 数据长度 | 受 URL 长度限制 | 受服务器配置和请求体限制 |
| 是否等于安全 | 不等于安全 | 不等于安全，仍需 HTTPS 和权限校验 |

参数放在请求体中不代表自动加密，密码和敏感数据仍必须使用 HTTPS 传输。

### 3.7 使用 HttpServletRequest 获取请求数据

Tomcat 会解析 HTTP 请求，并封装成 `HttpServletRequest` 对象，Spring MVC 再把它传入 Controller 方法。

~~~java
@RestController
public class RequestController {
    @RequestMapping("/request")
    public String request(HttpServletRequest request) {
        String name = request.getParameter("name");
        String uri = request.getRequestURI();
        String url = request.getRequestURL().toString();
        String method = request.getMethod();
        String userAgent = request.getHeader("User-Agent");

        System.out.println("name = " + name);
        System.out.println("uri = " + uri);
        System.out.println("url = " + url);
        System.out.println("method = " + method);
        System.out.println("userAgent = " + userAgent);
        return "request success";
    }
}
~~~

常用方法：

| 方法 | 作用 |
|---|---|
| `getParameter("name")` | 获取查询参数或表单参数 |
| `getRequestURI()` | 获取不含域名的请求路径 |
| `getRequestURL()` | 获取完整请求地址 |
| `getMethod()` | 获取 GET、POST 等请求方法 |
| `getHeader("User-Agent")` | 获取指定请求头 |

### 3.8 常见 HTTP 请求方法

| 方法 | 语义 | 是否应有请求体 | 是否幂等 |
|---|---|---|---|
| GET | 获取资源 | 通常没有 | 是 |
| POST | 提交数据、创建资源或触发处理 | 通常有 | 通常不是 |
| PUT | 用完整表示替换资源 | 可以有 | 是 |
| PATCH | 局部修改资源 | 可以有 | 视接口设计 |
| DELETE | 删除资源 | 通常没有 | 是 |

“幂等”表示同一个请求重复执行多次，最终资源状态与执行一次相同；它不表示每次请求都返回完全相同的响应。

## 4. HTTP 响应

### 4.1 响应的整体结构

服务器返回给浏览器的数据称为响应协议，包括：

1. 响应行
2. 响应头
3. 空行
4. 响应体

~~~text
响应行
响应头 1
响应头 2

响应体
~~~

### 4.2 响应行的定义和格式

响应行是响应的第一行，由协议版本、响应状态码和状态码描述组成：

~~~http
HTTP/1.1 200 OK
~~~

| 部分 | 含义 |
|---|---|
| 协议版本 | 服务器使用的 HTTP 版本 |
| 状态码 | 用三位数字表示处理结果 |
| 状态码描述 | 对状态码的文字说明 |

### 4.3 响应头的定义和常见字段

响应头从第二行开始，格式同样是 `key: value`，用于说明响应内容和缓存规则。

| 响应头 | 含义 |
|---|---|
| `Content-Type` | 响应体类型，例如 `text/html`、`application/json` |
| `Content-Length` | 响应体大小 |
| `Content-Encoding` | 响应体压缩方式，例如 `gzip` |
| `Cache-Control` | 客户端如何缓存响应 |
| `Set-Cookie` | 告诉浏览器保存 Cookie |
| `Location` | 重定向或新资源地址 |

### 4.4 响应体的定义

响应体是响应头后的最后一部分，保存真正返回的数据，例如 HTML、JSON、图片或文本。响应头和响应体之间必须有空行。

### 4.5 响应状态码

状态码的第一位表示大类：

| 范围 | 类别 | 说明 |
|---|---|---|
| 1xx | 信息 | 请求正在处理或需要继续操作 |
| 2xx | 成功 | 请求已成功处理 |
| 3xx | 重定向 | 需要访问其他地址或使用缓存 |
| 4xx | 客户端错误 | 请求参数、权限或资源存在问题 |
| 5xx | 服务端错误 | 服务器处理失败 |

常见状态码：

| 状态码 | 含义 | 常见场景 |
|---|---|---|
| 200 OK | 请求成功 | 查询或普通操作成功 |
| 201 Created | 创建成功 | 新增资源成功 |
| 204 No Content | 成功但没有响应体 | 删除成功且不返回数据 |
| 400 Bad Request | 请求参数错误 | 参数格式或校验失败 |
| 401 Unauthorized | 未认证 | 没有登录或令牌失效 |
| 403 Forbidden | 没有权限 | 已登录但无权访问 |
| 404 Not Found | 资源不存在 | 路径或数据不存在 |
| 405 Method Not Allowed | 请求方法不支持 | GET 地址使用了 POST |
| 500 Internal Server Error | 服务端异常 | 未处理的程序错误 |

状态码只表示处理结果，不应把所有业务错误都返回 200。前后端应约定状态码和统一业务响应格式。

### 4.6 使用 HttpServletResponse 设置响应

~~~java
@RestController
public class ResponseController {
    @RequestMapping("/response")
    public void response(HttpServletResponse response) throws IOException {
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.setHeader("name", "itcast");
        response.setContentType("text/html;charset=UTF-8");
        response.getWriter().write("<h1>hello response</h1>");
    }
}
~~~

更常见的写法是使用 `ResponseEntity`：

~~~java
@GetMapping("/response2")
public ResponseEntity<String> response2() {
    return ResponseEntity
            .status(HttpStatus.UNAUTHORIZED)
            .header("name", "itcast")
            .body("hello response");
}
~~~

没有特殊需求时，不建议手动设置响应头和状态码，Spring MVC 会根据处理结果自动生成。

## 5. Spring Boot Web 案例

### 5.1 Controller 返回 JSON

`@ResponseBody` 表示方法返回值直接写入响应体。返回实体对象或集合时，Spring Boot 会通过 Jackson 转换为 JSON。

~~~java
@Controller
public class UserController {
    @ResponseBody
    @GetMapping("/users")
    public List<User> list() {
        return userService.list();
    }
}
~~~

`@RestController` 等价于 `@Controller` 加上类级别的 `@ResponseBody`：

~~~java
@RestController
public class UserController {
    @GetMapping("/users")
    public List<User> list() {
        return userService.list();
    }
}
~~~

### 5.2 前后端请求流程

~~~mermaid
sequenceDiagram
    participant B as 浏览器
    participant C as Controller
    participant S as Service
    participant D as 数据访问层
    participant DB as 数据源
    B->>C: GET /users
    C->>S: 调用业务方法
    S->>D: 查询数据
    D->>DB: 读取数据
    DB-->>D: 返回记录
    D-->>S: 返回对象集合
    S-->>C: 返回业务结果
    C-->>B: JSON 响应
~~~

### 5.3 统一响应结果

统一响应对象让前端用同一套方式处理成功和失败：

~~~java
public record Result<T>(Integer code, String message, T data) {
    public static <T> Result<T> success(T data) {
        return new Result<>(1, "success", data);
    }
    public static Result<Void> error(String message) {
        return new Result<>(0, message, null);
    }
}
~~~

`data` 放业务数据，`message` 给出可读提示，`code` 表示业务结果。HTTP 状态码仍应表达认证、参数和服务器错误，不能用业务 `code` 代替所有 HTTP 状态码。

## 6. 三层架构与分层解耦

### 6.1 为什么要分层

如果 Controller 同时负责接收请求、读取文件、处理业务和拼接响应，代码会越来越长，修改一个功能可能影响其他功能。分层可以让每层只负责一类工作。

### 6.2 三层架构职责

| 层 | 常见包名 | 职责 |
|---|---|---|
| Controller | `controller` | 接收请求、参数校验、返回响应 |
| Service | `service` | 组织业务规则和事务 |
| DAO/Mapper | `dao`、`mapper` | 访问数据库或其他数据源 |

调用顺序通常是：

~~~mermaid
flowchart LR
    A[前端] --> B[Controller 控制层]
    B --> C[Service 业务层]
    C --> D[DAO/Mapper 数据访问层]
    D --> E[数据库]
~~~

三层架构的优点：职责清晰、代码复用、方便测试、便于替换实现和维护。

### 6.3 内聚和耦合

- 内聚：一个模块内部各个功能之间联系的紧密程度。
- 耦合：不同模块之间依赖和关联的程度。
- 高内聚：模块内部集中完成一类职责。
- 低耦合：模块之间尽量依赖抽象，而不是依赖具体实现。

例如 Controller 直接 `new UserServiceImpl()`，就会依赖具体实现。以后更换为 `UserServiceImpl2` 时必须修改 Controller，这就是紧耦合。

### 6.4 解耦的基本思路

1. 定义接口，例如 `UserService`。
2. 编写一个或多个实现类，例如 `UserServiceImpl`。
3. 不在 Controller 中直接 `new` 实现类。
4. 把对象交给 Spring 容器创建和管理。
5. Controller 只声明需要的接口，由容器注入实现对象。

## 7. IoC、DI 和 Bean

### 7.1 IoC 的定义

IoC 是 Inversion of Control，控制反转。对象原本由程序员在代码中主动 `new` 出来，使用 IoC 后，对象的创建、组装和生命周期控制权交给 Spring 容器。

IoC 解决的是“对象由谁创建和管理”的问题。

### 7.2 DI 的定义

DI 是 Dependency Injection，依赖注入。一个对象运行时需要另一个对象时，由 Spring 容器把所需对象传入，而不是由当前对象自己创建。

例如 `UserController` 依赖 `UserService`：

~~~text
没有 DI：UserController 自己 new UserServiceImpl()
使用 DI：Spring 容器创建 UserServiceImpl，再注入 UserController
~~~

### 7.3 Bean 的定义

Bean 是由 Spring IoC 容器创建、组装和管理的对象。加上组件注解只是声明 Bean 的一种方式，Bean 最终是否生效还取决于组件扫描范围和配置。

### 7.4 IoC 容器启动时做了什么

可以把容器理解为一个负责“登记和组装对象”的工厂。启动过程通常包括：

1. 读取配置类和组件扫描规则。
2. 找到 `@Component`、`@Service`、`@Controller` 等组件，生成 Bean 定义。
3. 创建 Bean 实例，并解析构造器、字段或 Setter 中的依赖。
4. 执行初始化回调，把完成组装的 Bean 放入容器。
5. Controller 接收请求时，从容器中使用已经准备好的对象。

代码中也可以显式声明 Bean：

~~~java
@Configuration
public class AppConfig {
    @Bean
    public Clock systemClock() {
        return Clock.systemDefaultZone();
    }
}
~~~

`@Bean` 方法的返回对象会注册到容器中，适合管理第三方类或需要自定义创建过程的对象。业务类通常使用组件注解即可。

### 7.5 Bean 的作用域和生命周期

默认作用域是 `singleton`，同一个容器中通常只有一个实例；`prototype` 每次获取时创建新实例。Web 项目还可以使用 request、session 等 Web 作用域，但必须考虑线程和生命周期。

~~~java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class TemporaryContext {
}
~~~

单例 Bean 会随容器启动创建并在容器关闭时销毁。不要在单例 Bean 中保存某个用户请求的可变数据，否则并发请求可能互相覆盖。

## 8. Bean 声明和组件扫描

### 8.1 四种常见组件注解

| 注解 | 作用 | 推荐使用位置 |
|---|---|---|
| `@Component` | 通用组件声明 | 不属于其他层的组件 |
| `@Controller` | 声明控制器 Bean | Controller 层 |
| `@Service` | 声明业务 Bean | Service 层 |
| `@Repository` | 声明数据访问 Bean | DAO 层 |

`@Controller`、`@Service` 和 `@Repository` 都是 `@Component` 的衍生注解，用于表达更清晰的分层语义。

~~~java
@Service
public class UserServiceImpl implements UserService {
}

@Repository
public class UserDaoImpl implements UserDao {
}
~~~

注解的 `value` 属性可以指定 Bean 名称。没有指定时，默认使用类名首字母小写，例如 `userServiceImpl`。

### 8.2 组件扫描

`@ComponentScan` 负责扫描组件注解并注册 Bean。`@SpringBootApplication` 内部包含组件扫描能力，默认扫描启动类所在包及其子包。

推荐的包结构：

~~~text
com.example
├── Application.java       启动类
├── controller
├── service
├── mapper
└── pojo
~~~

如果业务类放在启动类包之外，可能扫描不到，最终导致注入失败。解决方式是调整包结构或显式配置扫描范围。

扫描不到 Bean 时按顺序排查：类是否添加组件注解、启动类包是否是父包、是否被 profile 或条件配置排除、接口是否有多个实现，以及注入类型是否写错。

## 9. 依赖注入方式

### 9.1 @Autowired 按类型注入

`@Autowired` 默认按照类型从容器中查找依赖对象。如果容器中只有一个 `UserService` 实现，就可以直接注入。

### 9.2 属性注入

~~~java
@RestController
public class UserController {
    @Autowired
    private UserService userService;
}
~~~

优点是代码少，缺点是依赖隐藏在字段中、对象可能无法保持不可变，单元测试也不够直观。

### 9.3 构造器注入

~~~java
@RestController
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }
}
~~~

构造器注入能明确表达依赖，字段可以使用 `final`，对象创建后不容易被修改。如果类只有一个构造器，Spring 可以省略 `@Autowired`。实际项目优先推荐构造器注入。

### 9.4 Setter 注入

~~~java
@RestController
public class UserController {
    private UserService userService;

    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
}
~~~

Setter 注入适合可选依赖或需要在创建后重新设置的依赖，但会增加可变状态。

### 9.5 三种注入方式比较

| 方式 | 优点 | 缺点 | 常用建议 |
|---|---|---|---|
| 属性注入 | 简洁 | 隐藏依赖，不利于测试 | 老项目常见 |
| 构造器注入 | 依赖明确，可使用 `final` | 参数多时构造器较长 | 新项目优先 |
| Setter 注入 | 可选依赖更灵活 | 对象状态可变 | 特殊场景使用 |

## 10. 多个 Bean 的注入问题

### 10.1 为什么会报错

如果容器中有两个 `UserService` 实现，`@Autowired UserService userService` 只按类型查找就无法确定具体对象，会出现多个候选 Bean 的异常。

### 10.2 @Primary

给一个实现加上 `@Primary`，表示它是同类型 Bean 的默认选择：

~~~java
@Primary
@Service
public class UserServiceImpl implements UserService {
}
~~~

### 10.3 @Qualifier

`@Qualifier` 指定 Bean 名称，必须和 `@Autowired` 一起使用：

~~~java
@Autowired
@Qualifier("userServiceImpl")
private UserService userService;
~~~

### 10.4 @Resource

`@Resource` 是 Jakarta/JDK 体系提供的注解，常按名称查找 Bean：

~~~java
@Resource(name = "userServiceImpl")
private UserService userService;
~~~

### 10.5 注入注解对比

| 注解 | 提供方 | 默认查找规则 | 处理多个 Bean |
|---|---|---|---|
| `@Autowired` | Spring | 按类型 | 配合 `@Primary` 或 `@Qualifier` |
| `@Qualifier` | Spring | 指定名称 | 必须配合注入注解 |
| `@Resource` | Jakarta/JDK | 通常按名称 | 使用 `name` 指定 |

### 10.6 一个完整的依赖解析示例

~~~java
public interface NoticeSender {
    void send(String text);
}

@Service("emailSender")
public class EmailNoticeSender implements NoticeSender {
    public void send(String text) { /* 发送邮件 */ }
}

@Service
public class NoticeService {
    private final NoticeSender sender;

    public NoticeService(@Qualifier("emailSender") NoticeSender sender) {
        this.sender = sender;
    }

    public void notifyUser() {
        sender.send("操作成功");
    }
}
~~~

容器先注册 `EmailNoticeSender`，再创建 `NoticeService`；构造器参数需要 `NoticeSender`，`@Qualifier` 明确选择名为 `emailSender` 的 Bean。若缺少实现、扫描不到实现或名称拼写错误，启动阶段就会报依赖注入异常。

## 11. 本章总结

- Spring Boot 通过起步依赖、自动配置和内嵌 Tomcat 简化 Web 项目启动。
- HTTP 请求由请求行、请求头、空行和请求体组成；响应由响应行、响应头、空行和响应体组成。
- 请求行包含方法、资源路径和协议版本；请求头使用 `key: value` 描述客户端和内容信息；请求体保存提交数据。
- 响应状态码第一位表示类别，常见状态码包括 200、201、204、400、401、403、404 和 500。
- `@ResponseBody` 将返回值写入响应体，`@RestController` 等价于 `@Controller` 加 `@ResponseBody`。
- Controller、Service、DAO/Mapper 三层架构用于分离请求处理、业务逻辑和数据访问。
- IoC 把对象创建和管理交给 Spring 容器，DI 由容器向对象注入运行时依赖，Bean 是容器管理的对象。
- `@Component`、`@Controller`、`@Service`、`@Repository` 用于声明 Bean，组件必须位于扫描范围内。
- 依赖注入有属性、构造器和 Setter 三种方式，新项目优先使用构造器注入。
- 多个同类型 Bean 可以使用 `@Primary`、`@Qualifier` 或 `@Resource` 解决歧义。
