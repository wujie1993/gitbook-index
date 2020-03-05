# Thanos

## 项目简介

Thanos是一个基于Prometheus实现的监控方案，其主要设计目的是解决原生Prometheus上的痛点问题并且做进一步的提升，主要的特性有：全局查询，高可用，动态拓展，长期存储。

项目由Fabian Reinartz和Bartłomiej Płotka创建于2017年12月份。在2019年8月份成为CNCF沙盒项目。目前的维护成员主要来自于Red Hat。

## 官方网站

{% embed url="https://thanos.io/" %}

## 项目地址

{% embed url="https://github.com/thanos-io/thanos" %}

## 架构图

![](../../../.gitbook/assets/image%20%285%29.png)

## 组件介绍

* Sidecar。主要用于短期数据的查询和Prometheus本地数据上传。以gRPC的方式暴露StoreAPI接口，在Querier查询时将请求转为PromSQL代理到Prometheus并返回Querier。同时实现了本地文件的嗅探器，将Promethues所产生的本地只读块上传到远端对象存储中。
* Store。主要用于长期数据的查询。以gRPC的方式暴露StoreAPI接口，在Querier查询时读取远端对象存储中的数据并返回Querier。
* Compactor。定期地读取对象存储中的历史数据，进行下采样和压缩后保存回对象存储中，加速做大时间跨度查询时的速度。
* Receiver（实验性）。接收Prometheus的远程写入数据并上传到云存储中，支持写入时的租户隔离以及多副本写入。
* Ruler/Rule。通过Querier的Prometheus查询接口定期地获取指标并评估record和alert规则，将record规则的评估结果保存到本地，通过嗅探器将文件上传到远端对象存储中，将alert规则的评估结果用于触发Alertmanager告警。
* Querier/Query。作为指标查询入口，实现了Prometheus的查询接口，在接收到查询请求后通过StoreAPI转发请求到Sidecar和Store Gateway，将结果进行汇聚去重后返回客户端。在做大范围时间的指标查询时会通过自动下采样加速查询。同时自身也实现了StoreAPI，可处理来自于其他Querier的查询请求。Querier本身集成了与Prometheus类似的UI面板。
* Bucket。主要用于展示对象存储中历史数据的存储情况，查看每个指标源中数据块的压缩级别，解析度，存储时段和时间长度等信息

## 工作流程

### 指标写入

1. prometheus从所采集服务的metrics接口抓取指标数据，同时根据自身所配置的recording rules定期的对抓取到的指标数据进行评估，将结果以TSDB格式分块存储到本地，每个数据块的存储时长为2小时，且默认禁用了压缩功能。
2. sidecar嗅探到prometheus的数据存储目录生成了新的只读数据块（不包含head block）时，会将该数据块上传到对象存储桶中做为长期历史数据保存，在上传时会将数据块中的meta.json进行修改添加thanos相关的字段，如external\_labels。
3. rule根据所配置的recording rules定期地向query发起查询获取评估所需的指标值，并将结果以TSDB格式分块存储到本地。每个数据块的存储时长为2小时，且默认禁用了压缩功能，每个数据块的meta.json也附带了thanos拓展的external\_lables字段。当本地生成了新的只读数据块时，其自身会将该数据块上传到远端对象存储桶中做为长期历史数据保存。
4. compact定期地将对象存储中地数据块进行压缩和下采样，进行压缩时数据块中的truck会进行合并，对应的meta.json中的level也会一同增长，每次压缩累加1，初始值为1。在进行下采样时会创建新的数据块，根据采样步长从原有的数据块中抽取值存储到新的数据块中，在meta.json中记录resolution为采样步长。

### 指标读取

1. 客户端通过query API向query发起查询，query将请求转换成StoreAPI请求发送到其他的query，sidecar，rule和store上。
2. sidecar接收到来自于query发起的查询请求后将其转换成query API请求，发送给其绑定的prometheus，由prometheus从本地读取数据并响应，返回短期的本地采集和评估数据。
3. rule接收到来自于query发起的查询请求后直接从本地读取数据并响应，返回短期的本地评估数据。
4. store接收到来自于query发起的查询请求后首先从对象存储桶中遍历数据块的meta.json，根据其中记录的时间范围和标签先进行一次过滤。接下来从对象存储桶中读取数据块的index和chunks进行查询，部分查询频率较高的index会被缓存下来，下次查询使用到时可以直接读取。最终返回长期的历史采集和评估指标。

### 发送告警

1. prometheus根据自身配置的alerting规则定期地对自身采集的指标进行评估，当告警条件满足的情况下发起告警到alertmanager上。
2. rule根据自身配置的alerting规则定期的向query发起查询请求获取评估所需的指标，当告警条件满足的情况下发起告警到alertmanager上。
3. alertmanager接收到来自于prometheus和rule的告警消息后进行分组合并后发出告警通知

## 优势特性

* 统一查询入口——以Querier作为统一的查询入口，其自身实现了Prometheus的查询接口和StoreAPI，可为其他的Querier提供查询服务，在查询时会从每个Prometheus实例的Sidecar和Store Gateway获取到指标数据。
* 查询去重——每个数据块都会带有特定的集群标签，Querier在做查询时会去除集群标签，将指标名称和标签一致的序列根据时间排序合并。虽然指标数据来自不同的采集源，但是只会响应一份结果而不是多份重复的结果。
* 高空间利用率——每个Prometheus本身不存储长时间的数据，Sidecar会将Prometheus已经持久化的数据块上传到对象存储中。Compactor会定时将远端对象存储中的长期数据进行压缩，并且根据采样时长做清理，节约存储空间。
* 高可用——Querier是无状态服务，天生支持水平拓展和高可用。Store，Rule和Sidecar是有状态服务，在多副本部署的情况下也支持高可用，不过会产生数据冗余，需要牺牲存储空间。
* 存储长期数据——Prometheus实例的Sidecar会将本地数据上传到远端对象存储中作为长期数据
* 横向拓展——当Prometheus的指标采集压力过大时，可以创建新的Prometheus实例，将scrape job拆分给多个Prometheus，Querier从多个Prometheus查询汇聚结果，降低单个Prometheus的压力
* 跨集群查询——需要合并多个集群的查询结果时，仅需要在每个集群的Querier之上再添加一层Querier，这样的层层嵌套，可以使得集群规模无限制拓展。

## 快速开始

部署prometheus+sidecar

```text
export WORKSPACE=$(pwd)

# 获取项目
git clone --single-branch --branch v0.35.1 https://github.com/coreos/prometheus-operator.git
cd prometheus-operator

# 部署prometheus-operator
kubectl apply -f bundle.yaml
```

部署prometheus+sidecar

```text
# 部署prometheus与sidecar并配置prometheus指标采集
cd example/thanos
kubectl create ns thanos
sed -i 's/namespace: default/namespace: thanos/g' prometheus-role.yaml
sed -i 's/namespace: default/namespace: thanos/g' prometheus-role-binding.yaml
sed -i 's/namespace: default/namespace: thanos/g' prometheus.yaml
sed -i 's/namespace: default/namespace: thanos/g' prometheus-service.yaml
sed -i 's/namespace: default/namespace: thanos/g' sidecar-service.yaml
sed -i 's/namespace: default/namespace: thanos/g' prometheus-servicemonitor.yaml
kubectl apply -f prometheus-role.yaml -n thanos
kubectl apply -f prometheus-role-binding.yaml -n thanos
kubectl apply -f prometheus.yaml -n thanos
kubectl apply -f prometheus-service.yaml -n thanos
kubectl apply -f sidecar-service.yaml -n thanos
kubectl apply -f prometheus-servicemonitor.yaml -n thanos
```

部署alertmanager

```text
---
kind: Secret
apiVersion: v1
metadata:
  name: alertmanager-main
  namespace: default
data:
  alertmanager.yaml: >-
    Z2xvYmFsOgogIHJlc29sdmVfdGltZW91dDogMWgKcm91dGU6CiAgcmVjZWl2ZXI6IGRlZmF1bHQKICBncm91cF93YWl0OiAxMHMKICBncm91cF9pbnRlcnZhbDogMTBtCiAgZ3JvdXBfYnk6IFsnYWxlcnRuYW1lJ10KICByZXBlYXRfaW50ZXJ2YWw6IDMwbQogIHJvdXRlczogW10KcmVjZWl2ZXJzOgotIG5hbWU6IGRlZmF1bHQ=
type: Opaque
---
apiVersion: monitoring.coreos.com/v1
kind: Alertmanager
metadata:
  name: main
  namespace: default
spec:
  listenLocal: false
  imagePullPolicy: Always
  version: latest
  replicas: 1
---
kind: Service
apiVersion: v1
metadata:
  name: alertmanager-main
  namespace: default
  labels:
    alertmanager: main
spec:
  ports:
    - name: web
      protocol: TCP
      port: 9093
      targetPort: web
  selector:
    alertmanager: main
    app: alertmanager
```

配置thanos

{% code title="my-values.yaml" %}
```text
query:
  replicaCount: 3

# 启用bucket web
bucketweb:
  enabled: true

# 启用compactor
compactor:
  enabled: true

# 启用storegateway
storegateway:
  enabled: true

# 启用ruler
ruler:
  enabled: true
  serviceMonitor:
    enabled: true
  alertmanagers:
  - http://alertmanager-main.default:9093
  config:
    groups: []

# 配置存储桶
objstoreConfig:
  type: S3
  config:
    bucket: "thanos"
    endpoint: "minio-my-store.rook-minio:9000"
    region: "default"
    access_key: "TEMP_DEMO_ACCESS_KEY"
    insecure: true
    signature_version2: true
    encrypt_sse: false
    secret_key: "TEMP_DEMO_SECRET_KEY"
    put_user_metadata: {}
    http_config:
      idle_conn_timeout: 90s
      response_header_timeout: 2m
      insecure_skip_verify: false
    trace:
      enable: false
    part_size: 134217728
```
{% endcode %}

部署thanos

```text
cd $WORKSPACE
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-release bitnami/thanos --namespace cortex -f my-values.yaml
```

打开grafana添加prometheus数据源，配置thanos query访问地址： `http://my-release-thanos-querier.thanos:9090`

## 交互式教程

{% embed url="https://katacoda.com/bwplotka/courses/thanos" %}

