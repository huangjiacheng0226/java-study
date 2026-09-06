# 三、Linux 用户和权限

Linux 通过“用户—用户组—权限”管理系统资源。本篇重点掌握超级管理员 root、用户切换、用户与用户组管理、权限查看和权限修改。

## 1. root 用户和用户切换

### 1.1 root 与普通用户

Linux 是多用户系统。`root` 是权限最大的超级管理员；普通用户通常只在自己的 HOME 目录中拥有完整操作权限，离开 HOME 目录后多数位置只能读取或执行，不能随意修改。

| 用户类型 | 权限特点 | 使用建议 |
|---|---|---|
| root | 几乎可以执行所有管理操作 | 仅在需要时使用，避免误删系统文件 |
| 普通用户 | 日常操作安全，权限受到限制 | 作为日常工作的默认账号 |

### 1.2 su：切换用户

`su`（switch user）用于切换到指定用户。`-` 会同时加载目标用户的环境变量和 HOME 目录，通常建议使用。

```bash
su - 用户名       # 切换到指定用户并加载其环境
su -              # 省略用户名时切换到 root
exit              # 返回上一个用户
# 也可以按 Ctrl+D 返回
```

普通用户切换到其他用户通常需要输入密码；root 切换到其他用户不需要密码。不要长期使用 root 进行普通操作。

### 1.3 sudo：临时获得管理员权限

`sudo` 只为紧随其后的一条命令临时赋予 root 权限，前提是当前用户已配置 sudo 认证。

```bash
sudo apt update             # 仅本条命令以管理员权限执行
sudo systemctl restart ssh # 管理服务时常用 sudo
```

配置 sudo 的基本步骤（需要 root）：

```bash
su -
visudo                     # 安全地打开 /etc/sudoers
```

在文件末尾加入（把 `itheima` 换成实际用户名）：

```text
itheima ALL=(ALL) NOPASSWD:ALL  # 允许该用户 sudo，且不要求输入密码
```

保存并退出后，使用 `sudo 命令` 验证配置。生产环境应按需授予权限，避免给普通账号过大的管理范围。

## 2. 用户与用户组管理

### 2.1 用户组的作用

用户可以加入多个用户组。权限既可以针对单个用户设置，也可以针对用户所属的组设置，这样便于给一批用户授予相同的访问权限。

### 2.2 创建、删除和查询用户组

以下命令通常需要 root 权限：

```bash
groupadd developers       # 创建用户组
groupdel developers       # 删除用户组（组不再被使用时执行）
getent group              # 查看系统中的所有用户组
```

`getent group` 每行通常包含三部分：`组名称:组认证(x):组 ID`。

### 2.3 创建、删除和查询用户

```bash
useradd -g developers -d /home/alice alice  # 指定主组和 HOME 目录
useradd bob                                # 默认创建同名组，HOME 为 /home/bob
userdel bob                                # 删除用户，保留其 HOME 目录
userdel -r alice                           # 同时删除用户的 HOME 目录
id alice                                    # 查看 alice 的 UID、GID 和所属组
id                                          # 查看当前用户信息
usermod -aG developers bob                 # 将 bob 附加到 developers 组
getent passwd                              # 查看系统中的所有用户
```

| 命令 | 用途 | 关键选项或结果 |
|---|---|---|
| `useradd` | 创建用户 | `-g` 指定已有主组；`-d` 指定 HOME |
| `userdel` | 删除用户 | `-r` 连同 HOME 一起删除 |
| `id` | 查看身份信息 | 不带用户名时查看当前用户 |
| `usermod -aG` | 增加附加组 | `-a` 表示追加，避免覆盖原有组 |
| `getent passwd` | 查询用户库 | 字段为用户名、密码占位符、UID、GID、描述、HOME、登录 shell |

`getent passwd` 每行通常有 7 个字段：`用户名:密码(x):用户ID:组ID:描述:HOME目录:登录终端`。

## 3. 查看权限控制

### 3.1 使用 ls -l 查看权限

```bash
ls -l                    # 以列表形式显示权限、所有者和所属组
ls -l hello.txt          # 查看指定文件
```

典型结果中的权限字段如下：

```text
-rwxr-x---  1  alice  developers  128  Sep 6 10:00  hello.sh
```

| 部分 | 含义 |
|---|---|
| 第 1 个字符 | 文件类型：`-` 普通文件，`d` 目录，`l` 软链接 |
| 第 2～4 个字符 | 所属用户（user）的 `rwx` 权限 |
| 第 5～7 个字符 | 所属用户组（group）的 `rwx` 权限 |
| 第 8～10 个字符 | 其他用户（other）的 `rwx` 权限 |
| 后续字段 | 硬链接数、所属用户、所属组、大小、时间、名称 |

### 3.2 r、w、x 的含义

| 权限 | 普通文件 | 目录 |
|---|---|---|
| `r` 读 | 查看文件内容 | 查看目录中的文件名（如 `ls`） |
| `w` 写 | 修改文件内容 | 在目录中创建、删除、改名 |
| `x` 执行 | 将文件作为程序运行 | 进入目录或访问其内容（如 `cd`） |

例如 `drwxr-xr-x` 表示：这是目录；所属用户拥有 `rwx`，所属组拥有 `r-x`，其他用户拥有 `r-x`。`-` 表示对应权限不存在。

## 4. 修改权限和所有权

### 4.1 chmod：修改权限

只有文件所属用户或 root 可以修改权限。符号写法的格式为：

```bash
chmod [-R] 权限 文件或目录
```

```bash
chmod u=rwx,g=rx,o=x hello.txt  # u=用户，g=用户组，o=其他用户
chmod -R u=rwx,g=rx,o=x test     # -R 递归修改 test 目录及其内容
```

数字写法把 `r` 记为 4、`w` 记为 2、`x` 记为 1，再相加：

| 数字 | 权限 | 计算 |
|---:|---|---|
| 0 | `---` | 0 |
| 1 | `--x` | 1 |
| 2 | `-w-` | 2 |
| 3 | `-wx` | 2+1 |
| 4 | `r--` | 4 |
| 5 | `r-x` | 4+1 |
| 6 | `rw-` | 4+2 |
| 7 | `rwx` | 4+2+1 |

```bash
chmod 751 hello.txt  # 用户 rwx，组 r-x，其他用户 --x
```

### 4.2 chown：修改所属用户和用户组

`chown` 主要由 root 执行，用于修改文件或目录的所属用户、所属组。

```bash
chown [-R] 用户[:用户组] 文件或目录
```

```bash
chown alice hello.txt             # 修改所属用户为 alice
chown :developers hello.txt      # 只修改所属组为 developers
chown alice:developers hello.txt # 同时修改用户和用户组
chown -R alice:developers project # 递归修改目录及其内容
```

冒号 `:` 用于分隔用户和用户组；省略其中一侧即可只修改另一项。

## 5. 本篇知识点总结

1. `root` 是 Linux 超级管理员，普通操作应优先使用普通用户。
2. `su - 用户名` 用于切换用户，`exit` 或 `Ctrl+D` 返回上一个用户。
3. `sudo` 只对单条命令临时授权，使用前必须配置 sudoers。
4. 用户可以加入多个用户组，用户组便于批量管理权限。
5. `useradd`、`userdel`、`id`、`usermod`、`getent` 覆盖用户的创建、删除、查询和分组管理。
6. `ls -l` 的权限字段由文件类型以及 user、group、other 三组 `rwx` 组成。
7. 文件和目录中的 `r`、`w`、`x` 含义不同，目录权限尤其要结合“查看、修改、进入”理解。
8. `chmod` 修改权限，支持符号写法和 0～7 数字写法；`-R` 会递归影响目录内容。
9. `chown` 修改所属用户和用户组，通常需要 root 权限。
