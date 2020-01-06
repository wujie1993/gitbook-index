# Rook

## 官方网站

{% embed url="https://rook.io/" %}

## 项目地址

{% embed url="https://github.com/rook/rook" %}

## 快速开始

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
kubectl create -f cluster-test.yaml
```

安装命令行工具

```text
kubectl create -f toolbox.yaml
```

在toolbox中使用命令`ceph -s`查看集群状态

