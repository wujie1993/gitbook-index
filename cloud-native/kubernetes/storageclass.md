# StorageClass

## NFS

### 项目地址

{% embed url="https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner" %}

### 快速开始

环境准备：

* NFS 服务，假设已有`192.168.1.1:/data/nfs`
* Helm

1、添加 charts 仓库

```text
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
```

2、创建 release

```text
kubectl create ns nfs
helm install nfs-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --set nfs.server=192.168.1.1 \
  --set nfs.path=/data/nfs \
  --set image.repository=heegor/nfs-subdir-external-provisioner \
  --set storageClass.name=nfs-provisioner \
  --namespace nfs
```

