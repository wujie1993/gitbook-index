# Systemd

### 

### Q&A

#### 如何在systemd中source /etc/profile？

```text
[Unit]
Description=Run command with source /etc/profile

[Service]
Type=oneshot
RemainAfterExit=true
ExecStart=/bin/bash -c 'source /etc/profile && /bin/sh /root/test.sh'

[Install]
WantedBy=multi-user.target
```

#### 使用systemctl status命令查看不到日志？

重启systemd-journald服务

```text
systemctl restart systemd-journald.service
```

#### 如何同时输出日志到systemd和本地文件中？

程序直接将日志输出到控制台，在程序启动脚本中使用如下命令

```text
nohup {{ 程序路径 }} >> {{ 本地日志文件路径 }} 2>&1 &
```

在systemd配置中使用如下配置

```text
...

[Service]
# 使用forking模式可以截获启动脚本的子进程以及子进程的控制台输出
Type=forking
# 使用脚本做为启动命令
ExecStart={{ 脚本路径 }}

...
```

