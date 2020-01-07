# Rook

## 官方网站

{% embed url="https://rook.io/" %}

## 项目地址

{% embed url="https://github.com/rook/rook" %}

## 快速开始

| 硬盘符号 | 大小 | 作用 |
| :--- | :--- | :--- |
| sdb | 50GB | OSD Data |
| sdc | 50GB | OSD Data |
| sdd | 50GB | OSD Data |
| sde | 50GB | OSD Metadata |

{% hint style="info" %}
安装前使用命令`lvm lvs`,`lvm vgs`和`lvm pvs`检查上述硬盘是否已经被使用，若已经使用需要删除，且确保硬盘上不存在分区和文件系统
{% endhint %}

前置准备

```text
modprobe rbd
yum install -y lvm2
```

安装测试集群

```text
git clone --single-branch --branch release-1.2 https://github.com/rook/rook.git
cd rook/cluster/examples/kubernetes/ceph
kubectl create -f common.yaml
kubectl create -f operator.yaml
kubectl create -f cluster.yaml
```

安装命令行工具

```text
kubectl create -f toolbox.yaml
```

在toolbox中使用命令`ceph -s`查看集群状态

{% hint style="info" %}
在重装ceph集群时需要清理rook数据目录（默认：/var/lib/rook）
{% endhint %}

