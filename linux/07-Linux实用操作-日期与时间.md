# 四、Linux 实用操作（二）：日期、时区与系统时间

## 1. date 命令

### 1.1 查看和格式化

`date` 查看系统时间，`%` 标记控制格式。

~~~bash
date                         # 当前时间
date "+%Y-%m-%d %H:%M:%S"     # 年-月-日 时:分:秒
date "+%s"                    # Unix 时间戳
~~~

| 标记 | 含义 |
|---|---|
| `%Y` / `%y` | 四位 / 两位年份 |
| `%m` / `%d` | 月 / 日 |
| `%H` / `%M` / `%S` | 时 / 分 / 秒 |
| `%s` | Unix 时间戳 |

### 1.2 日期计算

~~~bash
date -d "+1 day" "+%Y-%m-%d"   # 明天
date -d "-7 days" "+%Y-%m-%d"  # 七天前
~~~

## 2. 设置时区

~~~bash
sudo rm -f /etc/localtime
sudo ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime  # 设置东八区
date "+%F %T %Z"                                             # 验证
~~~

## 3. 校准系统时间

~~~bash
sudo yum -y install ntp         # 安装 NTP
sudo systemctl start ntpd       # 启动同步服务
sudo systemctl enable ntpd      # 开机自启
sudo ntpdate -u ntp.aliyun.com  # 手动校准
~~~

## 4. 本篇知识点总结

1. `date` 查看和格式化时间。
2. `date -d` 支持日期计算。
3. 中国大陆常用 `Asia/Shanghai` 时区。
4. NTP 用于校准系统时间。
