# Redis 快速入门：基础认识与环境准备

## 1.1 Redis 是什么

Redis（Remote Dictionary Server）是一个开源的内存数据存储系统。它把数据主要保存在内存中，因此读写速度很快，同时也支持把数据持久化到磁盘。

Redis 常被用作：

- 缓存：减少数据库访问压力。
- 会话存储：保存登录状态、验证码等临时数据。
- 消息队列：在生产者和消费者之间传递消息。
- 排行榜和计数器：利用有序集合或原子自增实现。

## 1.2 Redis 的主要特点

| 特点 | 解释 | 常见用途 |
| --- | --- | --- |
| 内存读写 | 数据优先存放在内存中，访问延迟低 | 缓存、热点数据 |
| 丰富的数据类型 | 支持字符串、列表、集合、有序集合、哈希等 | 不同业务模型 |
| 持久化 | 可以将内存数据保存到磁盘 | 数据恢复 |
| 单线程命令执行 | 命令执行过程具有原子性，减少并发竞争 | 计数、库存扣减 |
| 高可用与集群 | 支持主从复制、哨兵和集群 | 分布式系统 |

## 1.3 安装 Redis

### 1.3.1 安装依赖

Redis 服务端运行前需要准备对应的操作系统环境。Windows 初学者可以使用 Redis 的兼容发行版或通过 WSL 安装 Linux；Linux 和 macOS 可以直接使用系统包管理器。

Linux（以 Ubuntu 为例）：

```bash
# 更新软件包索引
sudo apt update

# 安装 Redis 服务端和命令行客户端
sudo apt install redis-server redis-tools
```

macOS（使用 Homebrew）：

```bash
# 安装 Homebrew 后执行
brew install redis
```

Windows 如果使用 WSL，可以先在 WSL 中执行 Ubuntu 安装步骤。也可以安装 Redis 的 Windows 兼容版本，并确保 `redis-server.exe` 和 `redis-cli.exe` 所在目录已加入 PATH。

### 1.3.2 上传并解压安装包

在 Linux 中也可以下载源码压缩包后编译安装：

```bash
# 下载源码压缩包（版本号仅作示例）
wget https://download.redis.io/releases/redis-7.2.5.tar.gz

# 解压并进入目录
tar -zxvf redis-7.2.5.tar.gz
cd redis-7.2.5

# 编译 Redis
make
```

实际学习时优先使用包管理器安装，步骤更少；源码安装适合需要指定版本或研究编译过程的场景。

### 1.3.3 启动

```bash
# 直接启动 Redis 服务
redis-server

# Linux 使用 systemd 管理服务
sudo systemctl start redis-server
sudo systemctl enable redis-server
```

启动后默认监听 `6379` 端口。可以使用 `redis-cli ping` 检查服务是否正常。

## 1.4 Redis 客户端

### 1.4.1 命令行客户端

命令行客户端是学习 Redis 最直接的工具：

```bash
# 连接本机默认端口
redis-cli

# 指定主机和端口
redis-cli -h 127.0.0.1 -p 6379

# 测试连接，返回 PONG 表示成功
redis-cli ping
```

连接成功后会看到类似 `127.0.0.1:6379>` 的提示符。

### 1.4.2 图形化客户端

图形化客户端可以查看 key、数据类型、过期时间和服务器信息，适合观察数据，但不能替代命令行学习。常见工具包括 Redis Insight 等。

使用图形化客户端时通常填写：

| 配置项 | 示例 | 说明 |
| --- | --- | --- |
| Host | `127.0.0.1` | Redis 服务地址 |
| Port | `6379` | Redis 默认端口 |
| Username | 留空或配置值 | 启用 ACL 时填写 |
| Password | 留空或配置值 | 设置密码后填写 |

连接后可以新建 key、查看 value、执行命令和查看内存使用情况。

### 1.4.3 安装和使用

安装客户端后，先确认 Redis 服务已经启动，再使用命令行或图形化客户端连接。建议初学阶段先用命令行完成 `SET`、`GET`、`DEL`、`TTL` 等命令，再用图形化工具观察命令产生的数据。

## 1.5 Redis 与关系型数据库的区别

| 对比项 | Redis | MySQL 等关系型数据库 |
| --- | --- | --- |
| 主要存储位置 | 内存 | 磁盘 |
| 数据模型 | 键值与多种数据结构 | 表、行、列 |
| 查询方式 | 通过 key 和数据结构命令访问 | SQL 查询 |
| 访问速度 | 通常更快 | 通常更适合复杂查询 |
| 数据持久性 | 可配置，默认需要关注持久化策略 | 通常默认持久化 |
| 典型角色 | 缓存、计数、队列 | 核心业务数据 |

两者并不是互相替代的关系。实际项目中常见组合是：MySQL 保存最终数据，Redis 保存热点或临时数据。

## 1.6 第一个 Redis 命令

```text
127.0.0.1:6379> SET user:name "小明"
OK
127.0.0.1:6379> GET user:name
"小明"
```

`SET` 用来保存字符串，`GET` 用来读取字符串。Redis 中每条数据都通过唯一的 key 访问。

## 1.7 Key 的命名规范

建议使用“业务:对象:属性”的形式命名：

```text
user:1001:name
cart:1001:items
product:2001:stock
```

这种命名方式可以让 key 的归属和用途一眼可见，避免不同业务之间发生命名冲突。

## 1.8 本篇总结

1. Redis 是以内存读写为主的高性能键值数据库。
2. Redis 支持多种数据类型，适合缓存、计数、队列和排行榜等场景。
3. Redis 与 MySQL 常常配合使用，Redis 负责速度，MySQL 负责核心数据持久化。
4. `redis-server` 启动服务，`redis-cli` 连接服务。
5. 命令行客户端适合学习，图形化客户端适合观察和管理。
6. key 建议采用有层次的冒号命名法。
