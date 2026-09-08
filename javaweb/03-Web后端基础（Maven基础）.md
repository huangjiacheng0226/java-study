# 03 Web 后端基础（Maven 基础）

## 1. Maven 概述

### 1.1 Maven 解决的问题

Maven 用 POM 文件管理依赖、统一目录、执行测试、编译和打包。

POM 是 Maven 的项目对象模型；依赖通常先从本地仓库读取，缺少时再从远程仓库下载，插件负责编译、测试和打包等目标。默认构建输出目录是 target。

~~~text
src/main/java       主代码
src/main/resources  配置和资源
src/test/java       测试代码
pom.xml             项目描述文件
~~~

### 1.2 Maven 坐标

一个构件通常由 groupId、artifactId、version 唯一确定。

~~~xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
~~~

项目直接使用的依赖应直接声明，不要依赖偶然的传递依赖。

### 1.3 Maven 模型和仓库

Maven 项目由 `pom.xml` 描述，POM 中常见元素包括项目坐标、父工程、依赖、插件、属性和构建配置。仓库分为：

| 仓库 | 位置 | 作用 |
|---|---|---|
| 本地仓库 | 开发者电脑 | 缓存已下载的依赖和插件 |
| 中央仓库 | Maven 官方公共服务 | 提供常用开源构件 |
| 私服 | 公司内部服务器 | 缓存、审核和发布公司构件 |

依赖查找通常先检查本地仓库，本地没有时再从远程仓库下载。网络异常时可以检查仓库地址、代理和本地缓存。

### 1.4 安装和 IDEA 集成

安装 Maven 后配置 `MAVEN_HOME` 和 `PATH`，使用以下命令验证：

~~~text
mvn -v
~~~

IDEA 中可以在 Settings 的 Build Tools/Maven 中配置 Maven 主目录、用户配置文件和本地仓库。新手建议先使用项目统一的 Maven 配置，避免不同机器版本差异造成构建失败。

### 1.5 POM 常见配置

~~~xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>demo</artifactId>
  <version>1.0.0</version>
  <packaging>jar</packaging>
  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
  </properties>
</project>
~~~

`packaging` 默认是 `jar`，Web 项目也可以使用 `war`。`properties` 用于统一 Java 版本、依赖版本和插件参数。父 POM 可以继承公共配置，子模块只保留自身依赖。

POM 还可以配置 `<build>`、插件和资源目录。插件决定“如何编译、测试和打包”，依赖决定“代码运行时需要哪些库”，二者不要混淆。

## 2. 依赖管理

### 2.1 依赖范围

| scope | 编译 | 测试 | 运行 | 常见用途 |
|---|---|---|---|---|
| compile | 是 | 是 | 是 | 默认范围 |
| provided | 是 | 是 | 否 | 容器提供的 API |
| runtime | 否 | 是 | 是 | 运行时驱动 |
| test | 否 | 是 | 否 | JUnit、Mockito |
| import | 依赖管理 | 依赖管理 | 依赖管理 | BOM |

传递依赖会自动引入间接库。版本冲突可用显式版本、dependencyManagement 或 exclusions 解决。

建议使用父 POM 或 dependencyManagement 统一版本，不要把 system scope 当作常规依赖方案。升级依赖后执行测试并用 dependency:tree 检查实际版本。

依赖冲突时 Maven 会根据依赖路径选择实际版本。可以在依赖树中定位冲突，再使用显式版本或 `<exclusions>` 排除不需要的传递依赖：

~~~xml
<dependency>
  <groupId>com.example</groupId>
  <artifactId>legacy-client</artifactId>
  <version>1.0.0</version>
  <exclusions>
    <exclusion>
      <groupId>org.example</groupId>
      <artifactId>old-lib</artifactId>
    </exclusion>
  </exclusions>
</dependency>
~~~

### 2.2 常用命令

~~~text
mvn clean
mvn test
mvn package
mvn install
mvn dependency:tree
~~~

`clean` 删除 target，`compile` 编译主代码，`test` 执行测试，`package` 生成 jar 或 war，`install` 将构件安装到本地仓库，`dependency:tree` 查看依赖树。

### 2.3 POM 依赖写法

~~~xml
<properties>
  <java.version>17</java.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
</dependencies>
~~~

依赖的 `groupId`、`artifactId`、`version` 组成坐标。使用 Spring Boot 父 POM 或 BOM 时，很多版本可以由父工程统一管理。

`dependencyManagement` 只负责统一版本，不会自动把依赖加入项目；仍需在 `<dependencies>` 中声明实际使用的依赖。`import` 范围通常用于导入 BOM。

依赖排除只影响当前依赖路径，不会删除其他路径带来的同一构件。遇到 `ClassNotFoundException`、方法版本不匹配等问题，应先查看依赖树，再确认 scope 和实际打包内容。

## 3. 生命周期

Maven 有 clean、default、site 三条内置生命周期。执行后面的阶段，会先执行该生命周期的前置阶段。

常用 default 生命周期顺序是 `validate → compile → test → package → verify → install → deploy`。例如执行 `mvn package` 会先编译和测试，再生成可发布构件。

插件目标可以直接执行，例如 `mvn compiler:compile`、`mvn surefire:test`。实际项目通常通过生命周期绑定插件，不建议依赖某个开发者机器上的手动命令。

常用命令组合：`mvn clean package` 先清理旧产物再打包；`mvn test -DskipTests` 只编译测试但跳过执行（仅用于临时排错，不应作为质量验证）；`mvn package -DskipTests` 生成构件但跳过测试，发布前不要使用。

## 4. JUnit 单元测试

~~~java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

class CalculatorTest {
    @Test
    void addsTwoNumbers() {
        assertEquals(5, 2 + 3);
    }
}
~~~

测试应独立、可重复，并使用断言验证结果。

常见 JUnit 5 注解：

| 注解 | 作用 |
|---|---|
| `@Test` | 标记测试方法 |
| `@BeforeEach` | 每个测试前执行 |
| `@AfterEach` | 每个测试后执行 |
| `@BeforeAll` | 所有测试前执行一次 |
| `@DisplayName` | 设置测试名称 |
| `@Disabled` | 暂时跳过测试 |

常见断言包括 `assertEquals`、`assertNotNull`、`assertTrue`、`assertThrows`。测试应只验证一个清晰行为，失败时能快速定位原因。

测试类通常放在 `src/test/java`，命名为 `XxxTest`。测试资源放在 `src/test/resources`。执行 `mvn test` 会由 Surefire 插件发现并运行测试；执行 `mvn verify` 可以在测试后执行更多质量检查。

单元测试应遵循 Arrange（准备）、Act（执行）、Assert（断言）结构。不要依赖测试执行顺序，也不要连接真实生产数据库；需要外部依赖时使用测试数据库、Mock 或独立测试容器。

## 5. 前面章节小结

- Maven 管理依赖、构建和项目结构。
- 坐标、依赖范围、传递依赖和生命周期是重点。
- 遇到依赖问题先执行 dependency:tree。

## 6. Maven 高级项目组织

### 6.1 分模块设计

大型项目可以拆成 `pojo`、`mapper`、`service`、`web` 等模块。模块之间通过 Maven 依赖连接，职责清晰，修改和复用更方便；依赖方向应从上层业务模块指向下层公共模块，避免循环依赖。

### 6.2 继承与版本锁定

父 POM 用 `<parent>` 被子模块继承，适合统一 Java 版本、插件和依赖版本。`dependencyManagement` 只锁定版本，子模块仍要在 `<dependencies>` 中声明实际使用的依赖。

~~~xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-dependencies</artifactId>
      <version>3.3.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
~~~

### 6.3 聚合工程

聚合工程通常只有一个父 POM，通过 `<modules>` 列出子模块。执行 `mvn clean install` 时，Maven 会按依赖顺序构建全部模块，不必手工逐个进入目录执行。

~~~xml
<packaging>pom</packaging>
<modules>
  <module>pojo</module>
  <module>service</module>
  <module>web</module>
</modules>
~~~

继承解决“配置复用”，聚合解决“统一构建”，一个工程可以同时使用二者。

### 6.4 私服和常见问题

私服用于缓存中央仓库依赖、发布公司内部构件和控制访问权限。上传/下载私服前要在 `settings.xml` 配置镜像、服务器账号和仓库地址；账号密码应使用环境变量或密码管理工具，不能提交到 Git。构建失败时优先查看完整错误堆栈、网络代理、JDK/Maven 版本和本地仓库缓存。

## 7. 本章总结

- Maven 通过坐标、依赖和生命周期统一管理 Java 项目的构建过程。
- 依赖范围、传递依赖、冲突排除和 `dependencyManagement` 决定项目最终的类路径。
- 多模块项目用继承复用配置、用聚合统一构建；私服用于共享内部构件和缓存依赖。
- 日常排错从 `mvn dependency:tree`、完整错误堆栈和 JDK/Maven 版本开始。
