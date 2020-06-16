# Kind

## 简介

kind主要用于在本地测试和开发kubernetes，通过简单的命令即可快速创建和管理集群，所有组件都以容器方式运行，并且支持高可用集群部署。

## 项目地址

{% embed url="https://github.com/kubernetes-sigs/kind" %}

## 官方文档

{% embed url="https://kind.sigs.k8s.io/" %}

## 快速开始

在使用kind前，需要确保docker已经在本地安装完毕

以安装v0.8.1版本为例

```text
GO111MODULE="on" go get sigs.k8s.io/kind@v0.8.1
```

创建kubernetes集群

```text
kind create cluster
```

删除kubernetes集群

```text
kind delete cluster
```

根据源代码编译构建kubernetes集群

```text
git clone https://github.com/kubernetes/kubernetes.git $(go env GOPATH)/src/k8s.io/kubernetes
kind build node-image
kind create cluster --image kindest/node:latest
```

