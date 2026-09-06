# 四、Linux 实用操作（一）：快捷键、软件安装与服务

## 1. 终端快捷键

### 1.1 常用控制快捷键

| 快捷键 | 作用 | 场景 |
|---|---|---|
| `Ctrl+C` | 停止命令或取消输入 | 停止 `tail -f` |
| `Ctrl+D` | 退出登录或交互程序 | 退出 `su`、Python |
| `Ctrl+L` | 清屏 | 清理终端 |
| `Ctrl+A` / `Ctrl+E` | 光标到开头 / 结尾 | 编辑长命令 |
| `Ctrl+←/→` | 按单词移动 | 编辑路径 |

`Ctrl+D` 不能退出 vi/vim，应使用 `:q` 或 `:q!`。

~~~bash
tail -f /var/log/messages  # 持续查看日志
# 按 Ctrl+C 停止输出
clear                      # 与 Ctrl+L 相同
~~~

### 1.2 history：复用历史命令

~~~bash
history                    # 查看历史命令
!grep                      # 执行最近一条以 grep 开头的命令
~~~

按 `Ctrl+R` 输入关键词反向搜索；回车执行，左右键取出后编辑。

## 2. 软件安装：yum 与 apt

`yum` 用于 CentOS 等 RPM 系统，`apt` 用于 Ubuntu 等 Debian 系统，均可自动解决依赖。

~~~bash
yum [-y] install|remove|search 软件名
apt [-y] install|remove|search 软件名
~~~

| 操作 | yum | apt |
|---|---|---|
| 安装 | `sudo yum install nginx` | `sudo apt install nginx` |
| 卸载 | `sudo yum remove nginx` | `sudo apt remove nginx` |
| 搜索 | `yum search nginx` | `apt search nginx` |

~~~bash
sudo yum -y install nmap  # 自动确认安装
sudo apt update           # 刷新软件索引
sudo apt -y install curl  # 安装 curl
~~~

## 3. systemctl：管理系统服务

~~~bash
systemctl start|stop|status|enable|disable 服务名
~~~

| 子命令 | 作用 |
|---|---|
| `start` | 启动 |
| `stop` | 停止 |
| `status` | 查看状态 |
| `enable` | 开机自启 |
| `disable` | 取消开机自启 |

~~~bash
sudo systemctl status sshd   # 查看 SSH 服务
sudo systemctl restart sshd  # 重启服务
sudo systemctl enable sshd   # 设置开机自启
~~~

常见服务：`NetworkManager`、`network`、`firewalld`、`sshd`。

## 4. ln：创建软链接

软链接类似 Windows 快捷方式，`ls -l` 首字符为 `l`。

~~~bash
ln -s 被链接文件或目录 链接路径
ln -s /var/www/site ~/site-link  # 创建链接
rm ~/site-link                   # 只删除链接
~~~

## 5. 本篇知识点总结

1. `Ctrl+C` 停止命令，`Ctrl+D` 退出会话，`Ctrl+L` 清屏。
2. `history`、`!前缀`、`Ctrl+R` 复用历史命令。
3. CentOS 常用 `yum`，Ubuntu 常用 `apt`。
4. `systemctl` 管理服务运行和开机自启。
5. `ln -s` 创建软链接。
