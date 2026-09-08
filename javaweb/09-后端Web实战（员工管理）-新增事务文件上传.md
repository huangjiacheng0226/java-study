# 09 后端 Web 实战（员工管理：新增、事务和文件上传）

## 1. 新增员工

### 1.1 业务流程

员工基本信息和工作经历需要同时保存，属于一个业务整体。

新增员工接口通常接收基本信息和工作经历集合。处理顺序是先校验请求，再插入员工主表；数据库回填主键后，把主键作为工作经历表的外键批量插入。

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

默认情况下，Spring 对运行时异常回滚，受检异常不一定自动回滚，因此跨表写入常使用 `rollbackFor = Exception.class`。常见传播行为：`REQUIRED` 加入当前事务或新建事务，`REQUIRES_NEW` 暂停当前事务并新建事务，`SUPPORTS` 有事务就加入、没有就非事务执行。

@Transactional 应放在 Service 的 public 方法上，并通过 Spring 代理调用；同类内部直接调用可能绕过代理。事务内不要执行耗时的远程 OSS 上传，避免长时间占用数据库连接。

跨表写入失败时必须回滚已完成的写操作，否则会出现只有员工没有经历、或只有部分经历的脏数据。批量插入应确认集合为空时不会生成非法 SQL。

事务生效的前提是方法由 Spring 代理对象调用、数据库引擎支持事务（MySQL 常用 InnoDB），并且异常没有被方法内部吞掉。不要在事务方法中捕获异常后只打印日志却继续返回成功，否则 Spring 无法感知失败并回滚。

## 2. 文件上传

### 2.1 本地存储示例

~~~java
@PostMapping("/upload")
public Result<String> upload(@RequestParam MultipartFile image) throws IOException {
    if (image.isEmpty()) {
        throw new IllegalArgumentException("文件不能为空");
    }
    String ext = FilenameUtils.getExtension(image.getOriginalFilename());
    String fileName = UUID.randomUUID() + "." + ext;
    Path target = Paths.get("uploads", fileName);

    Files.createDirectories(target.getParent());
    image.transferTo(target);
    return Result.success("/uploads/" + fileName);
}
~~~

上传接口接收 `MultipartFile`，文件内容通过请求体传输，表单的 `Content-Type` 必须是 `multipart/form-data`。保存成功后通常只返回文件访问地址或业务文件 ID，不直接返回服务器绝对路径。

### 2.2 安全规范

- 限制文件大小、扩展名和 MIME 类型。
- 使用随机文件名，避免覆盖已有文件。
- 不信任原始文件名，防止路径穿越。
- 生产环境使用 OSS 等对象存储，并控制访问权限。

Spring Boot 可通过 spring.servlet.multipart.max-file-size 和 max-request-size 限制上传大小。上传接口应限制 Content-Type、扩展名、文件头，并考虑病毒扫描和访问权限。

文件名应由服务器生成，不能直接使用用户上传的原始文件名。扩展名校验不能代替文件头校验；还应防止路径穿越、脚本文件上传和未授权访问。多实例部署时，本地磁盘文件不会自动共享，应使用 OSS 或其他对象存储。

对象存储的一般流程是：申请临时访问凭证或使用服务端密钥、上传文件、保存对象地址或对象键、按权限返回访问地址。数据库事务和远程文件上传通常不放在同一个长事务中；上传失败时要清理已上传的孤儿文件。

文件上传接口的前端表单必须设置 `enctype="multipart/form-data"`。后端可用 `@RequestParam("image") MultipartFile image` 接收文件，用 `image.getSize()`、`getContentType()` 和文件头做校验。下载或访问文件时还要检查当前用户是否有权限。

## 3. 事务四大特性

| 特性 | 含义 |
|---|---|
| 原子性 | 全部成功或全部失败 |
| 一致性 | 数据满足业务和约束 |
| 隔离性 | 并发事务互不产生错误影响 |
| 持久性 | 提交结果能够保存 |

隔离性通过数据库隔离级别实现。常见并发问题包括脏读、不可重复读和幻读；初学阶段先理解“一个事务看到的数据是否会被另一个事务中途修改”，再学习不同隔离级别的差异。

## 4. 本章总结

- 跨表写入必须放在同一事务。
- 文件上传要校验类型、大小、文件名和权限。
- 本地磁盘适合学习，OSS 更适合多实例部署。
