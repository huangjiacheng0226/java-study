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

### 2.2 常用命令

~~~text
mvn clean
mvn test
mvn package
mvn install
mvn dependency:tree
~~~

## 3. 生命周期

Maven 有 clean、default、site 三条内置生命周期。执行后面的阶段，会先执行该生命周期的前置阶段。

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

## 5. 本章总结

- Maven 管理依赖、构建和项目结构。
- 坐标、依赖范围、传递依赖和生命周期是重点。
- 遇到依赖问题先执行 dependency:tree。
