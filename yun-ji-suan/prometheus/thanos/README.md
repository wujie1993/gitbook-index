# Thanos

## 官方网站

{% embed url="https://thanos.io/" %}

## 项目地址

{% embed url="https://github.com/thanos-io/thanos" %}

## 架构图

![](../../../.gitbook/assets/image%20%282%29.png)

## 组件介绍

thanos包括以下组件：

* Sidecar。主要用于短期数据的查询和Prometheus本地数据上传。以gRPC的方式暴露StoreAPI接口，在Querier查询时将请求转为PromSQL代理到Prometheus并返回Querier。同时实现了本地文件的嗅探器，将Promethues所产生的本地只读（非Head Block）数据上传到远端对象存储中。
* Store Gateway。主要用于长期数据的查询。以gRPC的方式暴露StoreAPI接口，在Querier查询时读取远端对象存储中的数据并返回Querier。
* Compactor。定期地读取远端对象存储中的数据，进行下采样和压缩后保存回对象存储中。
* Receiver。接收Prometheus的远程写入数据并上传到云存储中
* Ruler/Rule。通过Querier的Prometheus查询接口定期地获取指标并评估record和alert规则，将record规则的评估结果保存到本地，通过嗅探器将文件上传到远端对象存储中，将alert规则的评估结果用于触发Alertmanager告警。
* Querier/Query。作为指标查询入口，实现了Prometheus的查询接口，在接收到查询请求后通过StoreAPI转发请求到Sidecar和Store Gateway，将结果进行汇聚去重后返回客户端，同时自身也实现了StoreAPI，可处理来自于其他Querier的查询请求。Querier本身集成了与Prometheus类似的UI面板。

## 优势特性

* 统一查询入口——以Querier作为统一的查询入口，其自身实现了Prometheus的查询接口和StoreAPI，可为其他的Querier提供查询服务，在查询时会从每个Prometheus实例的Sidecar和Store Gateway获取到指标数据。
* 查询去重——每个Prometheus实例都会带有特定的集群标签（如：region=east,replica=0）。Prometheus会为每个采集到的指标添加该集群标签，当多个Prometheus实例采集到同一个指标源时，会生成除以上集群标签外完全一致的标签。Querier在启动时可添加多个`--query.replica-label {{标签名}}`参数，Querier在从多个Prometheus实例获取到查询结果后，可以忽略以上的集群标签，并匹配每个指标的名称和标签，将完全一致的指标进行去重。
* 高空间利用率——每个Prometheus都是以建议的单实例的方式运行，不会采集多份数据。Compactor会定时将远端对象存储中的长期数据进行压缩和降精度，节约存储空间。
* 高可用——Querier是无状态服务，可以进行水平拓展，在Prometheus实例出现故障时，仍然可以通过Store Gateway查询远端数据
* 存储长期数据——Prometheus实例的Sidecar会将本地数据上传到远端对象存储中作为长期数据
* 横向拓展——当Prometheus的指标采集压力过大时，可以创建新的prometheus实例，将scrape job拆分给多个prometheus，Querier从多个prometheus查询汇聚结果，降低单个prometheus的压力
* 跨集群查询——需要合并多个集群的查询结果时，仅需要在每个集群的Querier之上再添加一层Querier，像树一样，每个根节点作为所涵盖区域的查询入口

## 快速开始

```text
# 获取项目
git clone --single-branch --branch v0.35.1 https://github.com/coreos/prometheus-operator.git
cd prometheus-operator

# 部署prometheus-operator
kubectl create -f bundle.yaml

# 部署prometheus与sidecar
kubectl create -f prometheus-role.yaml
kubectl create -f prometheus-role-binding.yaml
kubectl create -f prometheus.yaml
kubectl create -f prometheus-service.yaml
kubectl create -f sidecar-service.yaml

# 添加prometheus指标采集规则
kubectl create -f prometheus-servicemonitor.yaml

# 部署query
kubectl create -f query-deployment.yaml
kubectl create -f query-service.yaml

# 部署store gateway

```

## Q&A

### querier以什么规则判断需要读取下采样的值

### querier是如何去重的

是取平均值还是其中的某一份

## 交互式教程

{% embed url="https://katacoda.com/bwplotka/courses/thanos" %}



