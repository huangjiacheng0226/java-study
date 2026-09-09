# MySQL管理

> 本章介绍系统数据库和常用客户端工具，帮助你完成查看、备份和恢复等管理任务。


## 1.系统数据库

Mysql数据库安装完成后，自带了四个数据库，具体作用如下：

|数据库|含义|
|---|---|
|mysql|存储MySQL服务器正常运行所需要的各种信息(时区、主从、用户、权限等)|
|information_schema|提供了访问数据库元数据的各种表和视图，包含数据库、表、字段类型及访问权限等|
|performance_schema|为MySQL服务器运行时状态提供了一个底层监控功能，主要用于收集数据库服务器性能参数|
|sys|包含了一系列方便 DBA和开发人员利用 performance_schema性能数据库进行性能调优和诊断的视图|

## 2.常用工具

### 2.1 mysql

该mysql不是指mysql服务，而是指mysql的客户端工具

```Properties
mysql [options] [database]

options选项:
-u[username]|--user=username : 指定用户名，如-uroot
-p[password]|--password[=password] : 指定用户密码，如-p
-h[host]|--host=host : 指定服务器IP或域名，如-h127.0.0.1
-P[port]|--port=port : 指定连接端口（P为大写），如-P3306
-e statement|--execute=statement : 在客户端执行SQL语句并退出（无需进入mysql系统），但前面要跟上操作的数据库名，SQL用双引号包裹，如mysql -uroot -p mysql -e "select * from user;"
```

### 2.2 mysqladmin

mysqladmin是一个执行管理操作的客户端程序，可以用它来检查服务器的配置和当前运行状态，创建并删除数据库等

```Properties
## 通过帮助文档查看选项：
mysqladmin --help

示例选项:
create dbname : 创建指定数据库，如mysqladmin -uroot -p create text
drop dbname : 删除指定数据库，如mysqladmin -uroot -p drop text
ping : 检查MySQL服务器是否正在运行，如mysqladmin -uroot -p ping
status : 查看服务器状态，如mysqladmin -uroot -p status
shutdown : 关闭MySQL服务器（高风险操作），如mysqladmin -uroot -p shutdown
```

### 2.3 mysqlbinlog

由于服务器生成的二进制文件以二进制格式保存，所以如果想要检查这些文本的文本格式，就会使用到mysqlbinlog日志管理工具（需要管理员身份）

```Properties
mysqlbinlog [options] log-files1 log-files2 ...

mysqlbinlog log-file

options选项:
-d dbname|--database=dbname : 指定数据库名称，只列出指定的数据库相关操作，如mysqlbinlog -d mysql binlog.000001
-o n|--offset=n : 忽略日志中的前n行数据，如mysqlbinlog -o 10 binlog.000001
-r filename|--result-file=filename : 将输出的文本日志格式写到目标文件filename中,filename可以用绝对路径，如
mysqlbinlog -s binlog.000001 -r "F:\login.000001"
-s|--short-form : 让输出内容更简洁，只输出必要的信息，如mysqlbinlog -s binlog.000001
--start-datetime : 读取指定开始时间之后的事件，如mysqlbinlog --start-datetime="2025-03-18 10:00:00" binlog.000001
--stop-datetime : 读取指定结束时间之前的时间，如mysqlbinlog --stop-datetime="2025-03-18 12:00:00" binlog.000001
--start-position : 指定从二进制日志的哪个位置开始读取事件,如mysqlbinlog --start-position=100 binlog.000001
--stop-position : 指定读取二进制日志时的结束位置，如mysqlbinlog --stop-position=200 binlog.000001
```

- 在使用mysqlbinlog之前，需要先定位到binlog.000001所在文件夹，否则后面的使用要用到绝对路径或相对路径

### 2.4 mysqlshow

客户端对象查找工具，用来很快地查看存在哪些数据库、数据库中的表、表中的列和或者索引

```Properties
mysqlshow [options] [db_name [table_name [col_name]]]

options选项:
--count : 显示数据库及表的统计信息（数据库、表均可以不指定）
-i : 显示指定数据库或者指定表的状态信息

示例:
## 查询每个数据库的表的数量及表中记录的数量
mysqlshow -uroot -p --count
## 查询mysql库中每个表中的字段数及行数
mysqlshow -uroot -p mysql --count
## 查询mysql库中user表的详细信息
mysqlshow -uroot -p mysql user --count
```

### 2.5 mysqldump

mysqldump客户端工具用来备份数据库或在不同数据库之间进行数据迁移。备份内容包含创建表、及插入表的SQL语句

对于主要使用 InnoDB 的业务库，逻辑备份通常使用 `--single-transaction`，让备份在一个一致性快照中读取，减少对在线写入的影响。该选项不等价于“所有存储引擎都能无锁备份”，非事务型表仍需结合业务停写或其他方案评估。

命令行密码建议只写 `-p`，让客户端交互式读取，不要把明文密码直接写在命令、脚本或 shell 历史中。更多选项见 [mysqldump](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html)。

```mermaid
flowchart LR
    A[确认备份范围和磁盘空间] --> B[mysqldump 导出 SQL 文件]
    B --> C[校验文件和备份日志]
    C --> D[目标库创建或选择数据库]
    D --> E[mysql 或 source 导入]
    E --> F[抽样查询并校验数据]
```

```Properties
## 目标文件filename.sql可以用绝对路径或相对路径
mysqldump [options] db_name[tables] > filename.sql                ##备份指定数据库中的部分或全部表到目标文件
mysqldump [options] --database|-B db1 [db2 db3 ...] > filename.sql                ##备份多个指定的数据库到目标文件
mysqldump [options] --all-databases|-A > filename.sql                ##备份 MySQL 服务器上的所有数据库到目标文件

options连接选项:
-u[username]|--user=username : 指定用户名
-p[password]|--password[=password] : 指定用户密码，如-p
-h[host]|--host=host : 指定服务器IP或域名，如-h127.0.0.1
-P[port]|--port=port : 指定连接端口（P为大写），如-P3306

options输出选项:
--add-drop-database : 在每个数据库创建语句前加上drop database语句
--add-drop-table : 在每个表创建语句前加上drop table语句，默认开启，不开启（--skip-add-drop-table）
-n|--no-create-db : 不包含数据库的创建语句
-t|--no-create-info : 不包含数据表的创建语句
-d|--no-data : 不包含数据
-T filename|--tab[=filename] : 自动生成两个文件，一个.sql文件，创建表结构的语句，一个.txt文件，数据文件,例如：
## 拷贝mysql数据库下的user表
mysqldump -u root -p --tab=/path/to/export/directory mysql user 或
mysqldump -u root -p -T /path/to/export/directory mysql user
```

**-T filename\|--tab[=filename]参数**：

- 生成的两个文件文件名和表名一致

- filename指文件存放路径，可以是绝对路径，也可以是相对路径，如果当前已定位到指定路径，可以不指定filename

- -T后必须跟filename，而--tab后可以不跟filename

### 2.6 mysqlimport/source

mysqlimport是客户端数据导入工具，用来导入mysqldump加-T参数后导出的文本文件

```Properties
mysqlimport [options] db_name textfile1 [textfile2 ...]

示例：mysqlimport -uroot -p text /tmp/city.txt
```

如果需要导入sql文件，可以使用mysql中的source命令

```Properties
## filename.sql可以使用路径
source filename.sql;
```
## 小练习

使用 `mysqldump` 导出一个演示数据库，再用 `mysql` 或 `source` 导入到另一个数据库中。
