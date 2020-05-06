# Etcd

### 项目地址

{% embed url="https://github.com/etcd-io/etcd" %}

### 官方文档

{% embed url="https://etcd.io/docs/" %}

### 快速上手

启动etcd

```text
docker run -d -p 2379:2379 --rm quay.io/coreos/etcd:v3.3.19 /usr/local/bin/etcd --listen-client-urls http://0.0.0.0:2379 --advertise-client-urls http://0.0.0.0:2379
```

列举所有key

```text
ETCDCTL_API=3 etcdctl get --prefix / --keys-only
```

查看资源占用

```text
ETCDCTL_API=3 etcdctl --write-out=table endpoint status
```

### Q&A

空间超额处理办法mvcc: database space exceeded

{% embed url="https://www.centos.bz/2018/06/etcd%E6%95%B0%E6%8D%AE%E5%BA%93%E5%BC%82%E5%B8%B8%EF%BC%9Amvcc-database-space-exceeded%E8%A7%A3%E5%86%B3/" %}



