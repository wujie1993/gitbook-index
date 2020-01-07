# Rook

## 官方网站

{% embed url="https://rook.io/" %}

## 项目地址

{% embed url="https://github.com/rook/rook" %}

## 快速开始

```text
git clone --single-branch --branch release-1.2 https://github.com/rook/rook.git
cd rook/cluster/examples/kubernetes/ceph
kubectl create -f common.yaml
kubectl create -f operator.yaml
kubectl create -f cluster-test.yaml
```



