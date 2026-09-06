# 四、Linux 实用操作（四）：网络传输与端口

## 1. ping：检查连通性

~~~bash
ping [-c 次数] IP或主机名
ping -c 4 www.baidu.com  # 发送 4 次
~~~

不加 `-c` 会持续发送，按 `Ctrl+C` 停止。

## 2. wget：下载文件

~~~bash
wget [-b] URL
wget https://example.com/file.zip     # 前台下载
wget -b https://example.com/file.zip  # 后台下载
tail -f wget-log                      # 查看后台进度
~~~

## 3. curl：发送 HTTP 请求

~~~bash
curl URL
curl -O https://example.com/file.zip  # 按远程文件名保存
curl cip.cc                           # 查询公网 IP
~~~

`wget` 偏重下载，`curl` 更适合发送请求并查看响应。

## 4. 端口基础

IP 定位计算机，端口进一步定位应用。虚拟端口范围为 0～65535。

| 范围 | 类型 | 示例 |
|---:|---|---|
| 1～1023 | 公认端口 | SSH 22、HTTPS 443 |
| 1024～49151 | 注册端口 | 应用服务 |
| 49152～65535 | 动态端口 | 临时连接 |

## 5. 查看端口占用

~~~bash
sudo yum -y install nmap
nmap 192.168.88.130       # 扫描开放端口
sudo yum -y install net-tools
netstat -anp | grep 6000  # 筛选端口和进程
~~~

`0.0.0.0:6000` 表示绑定所有网卡。

## 6. 本篇知识点总结

1. `ping -c` 检查指定次数的连通性。
2. `wget` 下载，`-b` 后台下载；`curl` 发送 HTTP 请求。
3. 端口定位具体应用，22 是常见 SSH 端口。
4. `nmap` 扫描端口，`netstat -anp | grep` 查看占用。
