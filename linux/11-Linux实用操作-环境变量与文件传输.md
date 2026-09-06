# 四、Linux 实用操作（六）：环境变量与文件传输

## 1. 环境变量基础

环境变量是操作系统保存的 Key-Value 信息。常见变量有 HOME、USER 和 PWD。

~~~bash
env              # 查看全部环境变量
echo $HOME       # 读取 HOME
echo ${PATH}ABC  # 用大括号明确变量边界
~~~

## 2. PATH：命令搜索路径

PATH 是由冒号分隔的目录列表。Shell 按顺序在这些目录中查找可执行程序。

~~~bash
echo $PATH
export PATH=$PATH:/home/itheima/myenv  # 追加目录并保留原 PATH
~~~

不要直接覆盖 PATH，否则常用命令可能无法找到。

## 3. 设置环境变量

### 3.1 临时设置

~~~bash
export MYNAME=example  # 当前 Shell 及其子进程有效
echo $MYNAME
~~~

### 3.2 永久设置

当前用户写入 ~/.bashrc，所有用户写入 /etc/profile，然后用 source 加载。

~~~bash
echo 'export MYNAME=example' >> ~/.bashrc  # 追加用户配置
source ~/.bashrc                          # 立即生效
echo $MYNAME                              # 验证
~~~

## 4. 文件上传与下载

### 4.1 FinalShell 图形方式

FinalShell 下方文件窗口可浏览 Linux 目录：右键远程文件可下载，将本地文件拖入目标目录可上传。本地文件通常保存在软件配置的 fsdownload 文件夹。

### 4.2 rz 与 sz

~~~bash
sudo yum -y install lrzsz  # 安装工具
rz                         # 从本地上传
sz file.txt                # 下载到本地
~~~

rz、sz 需要 FinalShell、SecureCRT、XShell 等终端软件支持。

## 5. 本篇知识点总结

1. env 查看环境变量，$变量名 读取值。
2. PATH 决定 Shell 查找命令的目录顺序。
3. export 设置变量；配置文件用于永久保存。
4. 追加 PATH 时要保留原 PATH。
5. FinalShell 可图形化传输，rz 上传、sz 下载。
