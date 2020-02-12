# Thanos

## 官方网站

{% embed url="https://thanos.io/" %}

## 项目地址

{% embed url="https://github.com/thanos-io/thanos" %}

## 架构图

![](../../../.gitbook/assets/image%20%282%29.png)

## 组件介绍

thanos包括以下组件：

* Sidecar。主要用于短期数据的查询和Prometheus本地数据上传。以gRPC的方式暴露StoreAPI接口，在Querier查询时将请求转为PromSQL代理到Prometheus并返回Querier。同时实现了本地文件的嗅探器，将Promethues所产生的本地只读块上传到远端对象存储中。
* Store。主要用于长期数据的查询。以gRPC的方式暴露StoreAPI接口，在Querier查询时读取远端对象存储中的数据并返回Querier。
* Compactor。定期地读取对象存储中的历史数据，进行下采样和压缩后保存回对象存储中，加速做大时间跨度查询时的速度。
* Receiver。接收Prometheus的远程写入数据并上传到云存储中
* Ruler/Rule。通过Querier的Prometheus查询接口定期地获取指标并评估record和alert规则，将record规则的评估结果保存到本地，通过嗅探器将文件上传到远端对象存储中，将alert规则的评估结果用于触发Alertmanager告警。
* Querier/Query。作为指标查询入口，实现了Prometheus的查询接口，在接收到查询请求后通过StoreAPI转发请求到Sidecar和Store Gateway，将结果进行汇聚去重后返回客户端。在做大范围时间的指标查询时会通过自动下采样加速查询。同时自身也实现了StoreAPI，可处理来自于其他Querier的查询请求。Querier本身集成了与Prometheus类似的UI面板。
* Bucket。主要用于展示对象存储中历史数据的存储情况，查看每个指标源中数据块的压缩级别，解析度，存储时段和时间长度等信息

## 工作流程

### 指标采集

### 指标评估

### 指标查询

### 发送告警

## 优势特性

* 统一查询入口——以Querier作为统一的查询入口，其自身实现了Prometheus的查询接口和StoreAPI，可为其他的Querier提供查询服务，在查询时会从每个Prometheus实例的Sidecar和Store Gateway获取到指标数据。
* 查询去重——每个数据块都会带有特定的集群标签，Querier在做查询时会去除集群标签，将指标名称和标签一致的序列根据时间排序合并。虽然指标数据来自不同的采集源，但是只会响应一份结果而不是多份重复的结果。
* 高空间利用率——每个Prometheus本身不存储长时间的数据，Sidecar会将Prometheus已经持久化的数据块上传到对象存储中。Compactor会定时将远端对象存储中的长期数据进行压缩，并且根据采样时长做清理，节约存储空间。
* 高可用——Querier是无状态服务，天生支持水平拓展和高可用。Store，Rule和Sidecar是有状态服务，在多副本部署的情况下也支持高可用，不过会产生数据冗余，需要牺牲存储空间。
* 存储长期数据——Prometheus实例的Sidecar会将本地数据上传到远端对象存储中作为长期数据
* 横向拓展——当Prometheus的指标采集压力过大时，可以创建新的Prometheus实例，将scrape job拆分给多个Prometheus，Querier从多个Prometheus查询汇聚结果，降低单个Prometheus的压力
* 跨集群查询——需要合并多个集群的查询结果时，仅需要在每个集群的Querier之上再添加一层Querier，这样的层层嵌套，可以使得集群规模无限制拓展。

## 快速开始

```text
export WORKSPACE=$(pwd)

# 获取项目
git clone --single-branch --branch v0.35.1 https://github.com/coreos/prometheus-operator.git
cd prometheus-operator

# 部署prometheus-operator
kubectl apply -f bundle.yaml

# 部署prometheus与sidecar并配置prometheus指标采集
sed -i 's/namespace: default/namespace: thanos/g' prometheus-role.yaml
sed -i 's/namespace: default/namespace: thanos/g' prometheus-role-binding.yaml
sed -i 's/namespace: default/namespace: thanos/g' prometheus.yaml
sed -i 's/namespace: default/namespace: thanos/g' prometheus-service.yaml
sed -i 's/namespace: default/namespace: thanos/g' sidecar-service.yaml
sed -i 's/namespace: default/namespace: thanos/g' prometheus-servicemonitor.yaml
kubectl apply -f prometheus-role.yaml
kubectl apply -f prometheus-role-binding.yaml
kubectl apply -f prometheus.yaml
kubectl apply -f prometheus-service.yaml
kubectl apply -f sidecar-service.yaml
kubectl apply -f prometheus-servicemonitor.yaml

cd $WORKSPACE
git clone https://github.com/thanos-io/kube-thanos.git
cd kube-thanos/examples/all/manifests

# 部署query并配置指标采集
# 编辑thanos-query-deployment.yaml
# 添加参数- --store=dnssrv+_grpc._tcp.thanos-sidecar.thanos.svc.cluster.local
kubectl apply -f thanos-query-deployment.yaml
kubectl apply -f thanos-query-service.yaml
kubectl apply -f thanos-query-serviceMonitor.yaml

# 部署store并配置指标采集
kubectl apply -f thanos-store-statefulSet.yaml
kubectl apply -f thanos-store-service.yaml
kubectl apply -f thanos-store-serviceMonitor.yaml

# 部署compact并配置指标采集
kubectl apply -f thanos-compact-statefulSet.yaml
kubectl apply -f thanos-compact-service.yaml
kubectl apply -f thanos-compact-serviceMonitor.yaml

# 部署rule并配置指标采集
kubectl apply -f thanos-rule-statefulSet.yaml
kubectl apply -f thanos-rule-service.yaml
kubectl apply -f thanos-rule-serviceMonitor.yaml

# 部署receive并配置指标采集
kubectl apply -f thanos-receive-statefulSet.yaml
kubectl apply -f thanos-receive-service.yaml
kubectl apply -f thanos-receive-serviceMonitor.yaml

# 部署bucket
kubectl apply -f thanos-bucket-deployment.yaml
kubectl apply -f thanos-bucket-service.yaml
```

## Q&A

### querier如何实现自动下采样的值

### store内存缓存和本地缓存有何区别

## 交互式教程

{% embed url="https://katacoda.com/bwplotka/courses/thanos" %}



