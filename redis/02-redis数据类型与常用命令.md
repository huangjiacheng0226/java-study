# Redis 快速入门：数据类型与常用命令

## 2.1 Redis 数据类型总览

| 类型 | 数据形态 | 适合场景 | 常用命令 |
| --- | --- | --- | --- |
| String | 一个字符串或数字 | 缓存、计数器、验证码 | `SET`、`GET`、`INCR` |
| Hash | 字段和值的集合 | 用户对象、商品属性 | `HSET`、`HGET`、`HGETALL` |
| List | 有序字符串列表 | 消息队列、时间线 | `LPUSH`、`RPUSH`、`LPOP` |
| Set | 无序且不重复的集合 | 标签、共同好友 | `SADD`、`SMEMBERS`、`SINTER` |
| Sorted Set | 带分数的有序集合 | 排行榜、延迟队列 | `ZADD`、`ZRANGE`、`ZSCORE` |

## 2.2 String：字符串

### 2.2.1 概念解释

String 是最基础的数据类型，值可以是文本，也可以是整数或浮点数。一个 key 对应一个 value。

### 2.2.2 使用示例

```text
# 保存和读取字符串
SET user:1001:name "小明"
GET user:1001:name

# 设置过期时间为 60 秒
SET login:code:1001 "938214" EX 60
TTL login:code:1001

# 原子自增，常用于访问量统计
SET article:1001:views 0
INCR article:1001:views
GET article:1001:views
```

`EX` 可以在写入时设置过期时间，`TTL` 查看剩余秒数。计数操作由 Redis 原子执行，适合高并发场景。

## 2.3 Hash：哈希

### 2.3.1 概念解释

Hash 适合保存一个对象的多个字段。它类似于编程语言中的字典或 JavaScript 对象。

### 2.3.2 使用示例

```text
# 保存用户对象的多个字段
HSET user:1001 name "小明" age 18 city "杭州"

# 读取一个字段
HGET user:1001 name

# 读取全部字段和值
HGETALL user:1001

# 删除某个字段
HDEL user:1001 city
```

## 2.4 List：列表

### 2.4.1 概念解释

List 是有序、可重复的数据集合。它支持从左端或右端插入和删除元素，因此可以实现简单队列或栈。

### 2.4.2 使用示例

```text
# 右侧加入消息
RPUSH queue:email "邮件1" "邮件2"

# 左侧取出一条消息，取出后元素会从列表删除
LPOP queue:email

# 查看列表长度
LLEN queue:email

# 查看下标 0 到 10 的元素，-1 表示最后一个元素
LRANGE queue:email 0 10
```

一个常见队列模型是生产者使用 `RPUSH`，消费者使用 `LPOP`。

## 2.5 Set：集合

### 2.5.1 概念解释

Set 中的元素无序且不能重复，适合表示“属于某个分类的成员”。

### 2.5.2 使用示例

```text
# 添加标签，重复添加不会产生重复元素
SADD user:1001:tags redis database redis

# 查看所有标签
SMEMBERS user:1001:tags

# 判断是否包含某个标签
SISMEMBER user:1001:tags redis

# 求两个用户的共同标签
SINTER user:1001:tags user:1002:tags
```

## 2.6 Sorted Set：有序集合

### 2.6.1 概念解释

Sorted Set 中每个成员都有一个 score，Redis 按 score 排序。成员本身不能重复，但 score 可以修改。

### 2.6.2 使用示例

```text
# 保存用户积分，数字是 score
ZADD leaderboard 120 user:1001 95 user:1002 150 user:1003

# 按分数从低到高查看，WITHSCORES 同时显示分数
ZRANGE leaderboard 0 -1 WITHSCORES

# 按分数从高到低查看
ZREVRANGE leaderboard 0 2 WITHSCORES

# 查看某个成员的分数
ZSCORE leaderboard user:1001
```

## 2.7 通用命令

```text
# 判断 key 是否存在
EXISTS user:1001

# 查看 key 的数据类型
TYPE user:1001

# 删除 key
DEL user:1001

# 设置过期时间（秒）
EXPIRE user:1001 300

# 查看剩余过期时间
TTL user:1001
```

删除和过期操作要谨慎，过期后数据无法通过 Redis 恢复。

## 2.8 本篇总结

1. String 适合单值、缓存和计数。
2. Hash 适合保存对象的多个字段。
3. List 适合队列和栈。
4. Set 适合去重和集合运算。
5. Sorted Set 适合排行榜和按分数排序的业务。
6. `EXISTS`、`TYPE`、`DEL`、`EXPIRE`、`TTL` 是常用通用命令。
