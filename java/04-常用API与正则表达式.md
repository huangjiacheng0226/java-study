# 常用 API 与正则表达式

本篇整理原笔记第十、十一章。API 是可直接调用的现成类，正则表达式用于描述文本模式。

## 1. 常用工具类

| 类 | 常用用途 |
| --- | --- |
| `Math` | 数学运算、取整、幂和随机数 |
| `System` | 标准输入输出、时间和数组复制 |
| `Runtime` | 访问当前 Java 运行时环境 |
| `Object` | 所有类的根类，提供 `toString`、`equals` 等方法 |
| `Objects` | 空值检查和对象比较 |
| `Arrays` | 数组排序、查找、复制和转字符串 |

重写 `equals` 时通常也要重写 `hashCode`，这样对象才能在哈希集合中正常工作。

## 2. 精确数值和包装类

浮点数不适合直接表示金额，金额计算使用 `BigDecimal`：

```java
import java.math.BigDecimal;

BigDecimal price = new BigDecimal("19.90");
BigDecimal quantity = new BigDecimal("2");
BigDecimal total = price.multiply(quantity);
```

`BigInteger` 用于超出 `long` 范围的整数。`Integer`、`Double` 等包装类把基本类型包装成对象，支持自动装箱和自动拆箱。

```java
Integer number = 10;       // 自动装箱
int value = number + 1;    // 自动拆箱
int parsed = Integer.parseInt("42");
```

## 3. 日期和时间

优先使用 JDK 8 之后的 `java.time` API：

| 类型 | 表示 |
| --- | --- |
| `LocalDate` | 日期 |
| `LocalTime` | 时间 |
| `LocalDateTime` | 本地日期时间 |
| `ZonedDateTime` | 带时区的日期时间 |
| `Instant` | 时间线上的时间戳 |
| `DateTimeFormatter` | 格式化与解析 |
| `Duration` | 时分秒级时间间隔 |
| `Period` | 年月日级时间间隔 |

```java
import java.time.LocalDate;
import java.time.Period;

LocalDate birthday = LocalDate.of(2000, 1, 1);
LocalDate today = LocalDate.now();
int years = Period.between(birthday, today).getYears();
```

旧的 `Date`、`Calendar` 和 `SimpleDateFormat` 在维护旧项目时仍会遇到，但新代码优先选择不可变的 `java.time` 类型。

## 4. 正则表达式

常见组成：

| 写法 | 含义 |
| --- | --- |
| `[abc]` | a、b、c 中任意一个 |
| `[^abc]` | 除 a、b、c 外的字符 |
| `\\d` | 数字 |
| `\\w` | 字母、数字或下划线 |
| `.` | 任意字符 |
| `+` | 一次或多次 |
| `*` | 零次或多次 |
| `?` | 零次或一次 |
| `{n,m}` | 至少 n 次，至多 m 次 |
| `^` / `$` | 开始 / 结束 |

Java 字符串中的反斜杠需要再次转义，例如正则 `\d+` 在 Java 中写成 `"\\d+"`。

```java
String phone = "13812345678";
boolean valid = phone.matches("1[3-9]\\d{9}");
```

捕获分组使用 `(pattern)`，非捕获分组使用 `(?:pattern)`。`matches` 要求整个字符串匹配；`find` 可以在文本中查找局部匹配。

## 5. API 使用步骤

```mermaid
flowchart LR
    A[明确需求] --> B[查找 API 文档]
    B --> C[确认参数和返回值]
    C --> D[编写最小示例]
    D --> E[处理边界与异常]
```

查 API 时优先确认：类是否需要导入、方法是否为静态、参数类型、返回值、是否会抛异常，以及对象是否可变。

## 6. 总结

常用 API 的重点不在背诵全部方法，而在于会查文档、会选择类型。正则表达式适合校验和提取结构化文本，但复杂规则应配合清晰的测试样例，避免写出无法维护的超长表达式。
