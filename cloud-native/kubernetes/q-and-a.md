# Q&A

### 删除命名空间时一直处于Terminating状态

{% embed url="https://github.com/kubernetes/kubernetes/issues/19317\#issuecomment-180003984" %}

比如删除命名空间test-ns

1. 导出命名空间

```text
kubectl get namespace test-ns -o json > test-ns.json
```

2. 编辑命名空间，删除其中的.spec.finalizers字段

```text
vi test-ns.json
```

3. 开启另一各终端

```text
kubectl proxy
```

4. 更新命名空间

```text
curl -k -H "Content-Type: application/json" -X PUT --data-binary @test-ns.json http://127.0.0.1:8001/api/v1/namespaces/test-ns/finalize
```

### 如何避免并发更新资源对象导致数据错误

在更新资源对象时，带上resourceVersion字段

### 证书过期如何续签

[https://www.qikqiak.com/post/update-k8s-10y-expire-certs/](https://www.qikqiak.com/post/update-k8s-10y-expire-certs/)

