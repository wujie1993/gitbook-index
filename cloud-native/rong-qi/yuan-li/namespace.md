# Namespace

容器的资源隔离依赖于Linux内核所实现的各种命名空间，当前支持的命名空间类型如下：

| 命名空间 | 对应参数 | 隔离资源 | Linux内核版本 |
| :--- | :--- | :--- | :--- |
| Mount | CLONE\_NEWNS | 文件系统挂载点 | 2.4.19 |
| UTS | CLONE\_NEWUTS | 主机名与域名 | 2.6.19 |
| IPC | CLONE\_NEWIPC | 信号量、信息队列与共享内存 | 2.6.19 |
| PID | CLONE\_NEWPID | 进程编号 | 2.6.24 |
| Network | CLONE\_NEWNET | 网络设备、网络栈、端口等 | 2.6.29 |
| User | CLONE\_NEWUSER | 用户与用户组 | 3.8 |

Docker在创建容器进程时调用clone\(\)系统函数，并传递上述命名空间资源的对应参数。

