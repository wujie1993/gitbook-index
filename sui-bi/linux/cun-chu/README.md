# 存储

## Q&A

Q：umount 卸载挂载点时报 device busy（设备繁忙） 错误

A：首先通过 `yum install psmisc` 命令安装 fuser 工具，再使用 `fuser <挂载点路径>` 命令查询挂载点所占用的进程号，使用 kill 命令终止占用设备的进程号。

