# 四、Linux 实用操作（五）：进程与主机状态

## 1. 进程基础

程序运行后会被登记为进程，并获得唯一 PID。进程还可能有父进程 PPID。

## 2. ps：查看进程

~~~bash
ps -ef                 # 查看全部进程并显示完整信息
ps -ef | grep tail     # 筛选 tail 相关进程
ps -ef | grep 30001    # 按关键字或 PID 筛选
~~~

grep 自身也会出现在结果中，通常可以忽略。

| 字段 | 含义 |
|---|---|
| UID | 所属用户 ID |
| PID / PPID | 进程 ID / 父进程 ID |
| C | CPU 占用率 |
| STIME | 启动时间 |
| TTY | 所属终端，? 表示非终端启动 |
| TIME | 占用 CPU 的时间 |
| CMD | 启动命令或程序路径 |

## 3. kill：关闭进程

~~~bash
kill PID       # 请求进程正常退出
kill -9 PID    # 强制结束无响应进程
~~~

优先使用普通 kill，让程序自行清理资源。

## 4. top：实时查看状态

~~~bash
top            # 默认约每 5 秒刷新
~~~

按 q 或 Ctrl+C 退出。常见交互键：P 按 CPU 排序，M 按内存排序，k 输入 PID 结束进程，1 展开 CPU 核心。

| 字段 | 含义 |
|---|---|
| PID / USER | 进程 ID / 所属用户 |
| PR / NI | 优先级 / nice 值 |
| VIRT / RES / SHR | 虚拟、物理、共享内存 |
| S | S 休眠、R 运行、Z 僵尸、I 空闲 |
| %CPU / %MEM | CPU / 内存占用率 |
| TIME+ | 累计 CPU 时间 |

## 5. df、iostat、sar

### 5.1 df：磁盘空间

~~~bash
df -h  # 用人性化单位显示空间
~~~

### 5.2 iostat：CPU 与磁盘 I/O

~~~bash
iostat
iostat -x 2 3  # 每 2 秒刷新，共 3 次详细统计
~~~

重点指标：rKB/s、wKB/s 是读写速率，await 是平均等待时间，avgqu-sz 是平均队列长度，%util 是设备利用率。

### 5.3 sar：网络统计

~~~bash
sar -n DEV 3 2  # 每 3 秒采样一次，共 2 次
~~~

| 字段 | 含义 |
|---|---|
| IFACE | 网卡接口 |
| rxpck/s、txpck/s | 每秒收、发数据包数 |
| rxKB/s、txKB/s | 每秒收、发数据量 |
| rxcmp/s、txcmp/s | 每秒收、发压缩包数 |
| rxmcst/s | 每秒接收多播包数 |

## 6. 本篇知识点总结

1. 进程有 PID，父子进程通过 PPID 关联。
2. ps -ef 配合 grep 可筛选进程。
3. kill 请求退出，kill -9 强制结束。
4. top 实时观察 CPU、内存和进程。
5. df 看空间，iostat 看 I/O，sar 看网络。
