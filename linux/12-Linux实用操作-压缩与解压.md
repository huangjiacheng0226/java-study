# 四、Linux 实用操作（七）：压缩与解压

## 1. 常见格式

| 格式 | 特点与常见系统 |
|---|---|
| .tar | 主要打包归档，体积减少有限 |
| .tar.gz | tar 后使用 gzip 压缩，Linux/macOS 常用 |
| .zip | Linux、Windows、macOS 通用 |
| .7z、.rar | Windows 中常见 |

## 2. tar 命令

### 2.1 选项含义

~~~bash
tar [-c -v -x -f -z -C] 参数...
~~~

| 选项 | 含义 |
|---|---|
| -c | 创建归档 |
| -v | 显示处理过程 |
| -x | 解压 |
| -f | 指定归档文件，通常放在选项末尾 |
| -z | 使用 gzip |
| -C | 指定解压目录 |

### 2.2 示例

~~~bash
tar -cvf test.tar 1.txt 2.txt 3.txt             # 创建 tar
tar -zcvf test.tar.gz 1.txt 2.txt 3.txt         # 创建 tar.gz
tar -xvf test.tar                               # 解压到当前目录
tar -xvf test.tar -C /home/itheima             # 解压到指定目录
tar -zxvf test.tar.gz -C /home/itheima         # 解压 tar.gz
~~~

## 3. zip 与 unzip

~~~bash
zip -r project.zip project/        # 目录必须加 -r
unzip project.zip                  # 解压到当前目录
unzip -d /tmp/project project.zip  # 解压到指定目录
~~~

| 命令 | 用途 | 关键选项 |
|---|---|---|
| zip | 创建 zip 文件 | -r 递归处理目录 |
| unzip | 解压 zip 文件 | -d 指定输出目录 |

## 4. 本篇知识点总结

1. .tar 主要归档，.tar.gz 还使用 gzip 压缩。
2. tar 压缩常用 -cvf 或 -zcvf，解压常用 -xvf 或 -zxvf。
3. -f 指定归档文件，-C 指定解压目录。
4. zip 处理目录加 -r，unzip 用 -d 指定目标目录。
