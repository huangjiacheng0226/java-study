# 四、Linux 实用操作（三）：IP、主机名、域名解析与固定 IP

## 1. IP 地址

### 1.1 IPv4 与查看地址

IPv4 由四个 0～255 的数字组成。`ifconfig` 查看网卡信息。

~~~bash
ifconfig                      # 查看网卡和 IPv4
sudo yum -y install net-tools # 安装 ifconfig
~~~

### 1.2 特殊 IP

| 地址 | 含义 |
|---|---|
| `127.0.0.1` | 本机回环地址 |
| `0.0.0.0` | 所有本地网卡或任意来源 |

## 2. 主机名

~~~bash
hostname                              # 查看主机名
sudo hostnamectl set-hostname web01  # 修改主机名
~~~

## 3. 域名解析

系统先查询 hosts，再向 DNS 查询。

| 系统 | hosts 文件 |
|---|---|
| Windows | `C:\Windows\System32\drivers\etc\hosts` |
| Linux | `/etc/hosts` |

~~~text
192.168.88.130  centos  # 主机名映射
~~~

~~~bash
ssh itheima@centos  # 使用主机名连接
~~~

## 4. 配置固定 IP

DHCP 动态分配地址，重启后可能变化。VMware 网段、网关和 Linux 配置必须一致。

~~~ini
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.88.130       # 选择未占用地址
NETMASK=255.255.255.0
GATEWAY=192.168.88.2
DNS1=114.114.114.114
~~~

~~~bash
sudo systemctl restart network  # 重启网络
ifconfig                       # 验证 IP
ping -c 3 192.168.88.2         # 检查网关
~~~

## 5. 本篇知识点总结

1. IPv4 地址由四段 0～255 数字组成。
2. `127.0.0.1` 表示本机，`0.0.0.0` 常表示所有地址。
3. `hostname` 查看名称，`hostnamectl` 修改名称。
4. 域名解析先查 hosts，再查询 DNS。
5. 固定 IP 要匹配 VMware 网段、网关和网卡配置。
