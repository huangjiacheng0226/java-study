# 09 后端 Web 实战（员工管理：新增、事务和文件上传）

## 1. 新增员工

### 1.1 业务流程

员工基本信息和工作经历需要同时保存，属于一个业务整体。

~~~mermaid
flowchart TD
    A[接收员工 JSON] --> B[参数校验]
    B --> C[插入 emp]
    C --> D[取得员工主键]
    D --> E[批量插入工作经历]
    E --> F[提交事务]
    C -->|失败| X[回滚]
    E -->|失败| X
~~~

### 1.2 事务代码

~~~java
@Transactional(rollbackFor = Exception.class)
public void add(Emp emp) {
    empMapper.insert(emp); // 插入后回填主键
    if (emp.getExprList() != null) {
        exprMapper.insertBatch(emp.getId(), emp.getExprList());
    }
}
~~~

rollbackFor 明确哪些异常触发回滚；propagation 控制事务传播，常见 REQUIRED 表示加入现有事务或新建事务。

@Transactional 应放在 Service 的 public 方法上，并通过 Spring 代理调用；同类内部直接调用可能绕过代理。事务内不要执行耗时的远程 OSS 上传，避免长时间占用数据库连接。

## 2. 文件上传

### 2.1 本地存储示例

~~~java
@PostMapping("/upload")
public Result<String> upload(@RequestParam MultipartFile image) throws IOException {
    String ext = FilenameUtils.getExtension(image.getOriginalFilename());
    String fileName = UUID.randomUUID() + "." + ext;
    Path target = Paths.get("uploads", fileName);

    Files.createDirectories(target.getParent());
    image.transferTo(target);
    return Result.success("/uploads/" + fileName);
}
~~~

### 2.2 安全规范

- 限制文件大小、扩展名和 MIME 类型。
- 使用随机文件名，避免覆盖已有文件。
- 不信任原始文件名，防止路径穿越。
- 生产环境使用 OSS 等对象存储，并控制访问权限。

Spring Boot 可通过 spring.servlet.multipart.max-file-size 和 max-request-size 限制上传大小。上传接口应限制 Content-Type、扩展名、文件头，并考虑病毒扫描和访问权限。

## 3. 事务四大特性

| 特性 | 含义 |
|---|---|
| 原子性 | 全部成功或全部失败 |
| 一致性 | 数据满足业务和约束 |
| 隔离性 | 并发事务互不产生错误影响 |
| 持久性 | 提交结果能够保存 |

## 4. 本章总结

- 跨表写入必须放在同一事务。
- 文件上传要校验类型、大小、文件名和权限。
- 本地磁盘适合学习，OSS 更适合多实例部署。
