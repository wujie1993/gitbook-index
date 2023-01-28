# Q\&A

### 删除命名空间时一直处于Terminating状态

{% embed url="https://github.com/kubernetes/kubernetes/issues/19317#issuecomment-180003984" %}

比如删除命名空间test-ns

```
export ns=test-ns
kubectl get namespace ${ns} -o json   | tr -d "\n" | sed "s/\"finalizers\": \[[^]]\+\]/\"finalizers\": []/"   | kubectl replace --raw /api/v1/namespaces/${ns}/finalize -f -
```

### 如何避免并发更新资源对象导致数据错误

在更新资源对象时，带上resourceVersion字段

### 证书过期如何续签

{% embed url="https://www.qikqiak.com/post/update-k8s-10y-expire-certs/" %}

### ETCD集群地址变更

{% embed url="https://pytimer.github.io/2019/05/change-etcd-cluster-member-ip/" %}

### 如何开启防火墙并保持集群内部的正常网络通讯

在 cni 使用 overlay 网络模式和 kube-proxy 使用 iptables 进行请求转发时，无需额外配置策略，按照先重启 firewalld，再重启 kubelet 的顺序即可。

在 cni 使用路由直联模式时，除了重启 firewalld + kubelet 之外，需要额外添加白名单策略，允许来自 pod cidr 端的入栈请求

```
firewall-cmd --zone=trusted --add-source=<pod cidr> --permanent
firewall-cmd --zone=trusted --add-source=<pod cidr>
```

在 kube-proxy 使用 ipvs 进行请求转发时，除了重启 firewalld + kubelet 之外，需要额外添加白名单策略，允许地址转发请求

```
firewall-cmd --zone=public --add-masquerade --permanent
firewall-cmd --zone=public --add-masquerade
```

在开启了 NodeLocal DNSCache 的情况下，需要额外将集群 dns 地址加入白名单中

```
firewall-cmd --zone=trusted --add-source=<cluster dns> --permanent
firewall-cmd --zone=trusted --add-source=<cluster dns>
```
