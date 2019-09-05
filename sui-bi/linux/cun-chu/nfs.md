# NFS

## 服务端

安装

```bash
yum install nfs-utils -y
```

设置 NFS 服务开机启动

```bash
systemctl enable rpcbind
systemctl enable nfs
```

启动 NFS 服务

```bash
systemctl start rpcbind
systemctl start nfs
```

开启防火墙

```bash
firewall-cmd --zone=public --permanent --add-service={rpc-bind,mountd,nfs}
firewall-cmd --reload
# 通过iptables开启时，将以下命令列出的端口以及111和2049加入白名单
rpcinfo -p
```

配置共享目录

```text
mkdir /data
chmod 755 /data
```

配置导出目录

{% code-tabs %}
{% code-tabs-item title="/etc/exports" %}
```text
/data/     10.192.31.0/24(rw,sync,no_root_squash,no_all_squash)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

1. /data: 共享目录位置。
2. 192.168.0.0/24: 客户端 IP 范围，\* 代表所有，即没有限制。
3. rw: 权限设置，可读可写。
4. sync: 同步共享目录。
5. no\_root\_squash: 可以使用 root 授权。
6. no\_all\_squash: 可以使用普通用户授权。

重启nfs生效

```text
systemctl restart nfs
```

查看本地共享目录

```text
showmount -e localhost
```

## 客户端

安装

```text
yum install nfs-utils -y
```

开启rpcbind并设置开机自启动

```text
systemctl start rpcbind
systemctl enable rpcbind
```

查看服务端共享目录

```text
showmount -e 10.192.31.116
```

创建挂载目录

```text
mkdir /data
```

挂载nfs

```text
mount -t nfs 10.192.31.113:/data /data
```

配置开机自动挂载

{% code-tabs %}
{% code-tabs-item title="/etc/fstab" %}
```text
10.192.31.113:/data     /data                   nfs     defaults        0 0
```
{% endcode-tabs-item %}
{% endcode-tabs %}

