# systemd

### Q&A

#### 如何在systemd中加载/etc/profile中的环境变量？

```text
[Service]
...
# 在启动命令前使用source /etc/profile指令加载环境变量
ExecStart=/bin/bash -c 'source /etc/profile && {{ 启动命令 }}'
...
```

#### 使用systemctl status命令查看不到日志？

重启systemd-journald服务

```text
systemctl restart systemd-journald.service
```

#### 如何将日志输出到本地文件中？

程序直接将日志输出到控制台，在程序启动脚本中使用如下命令

```text
nohup {{ 程序路径 }} >> {{ 本地日志文件路径 }} 2>&1 &
```

在systemd配置中使用如下配置

```text
...

[Service]
# 使用forking模式可以截获启动脚本的子进程
Type=forking
# 使用脚本做为启动命令
ExecStart={{ 脚本路径 }}

...
```

