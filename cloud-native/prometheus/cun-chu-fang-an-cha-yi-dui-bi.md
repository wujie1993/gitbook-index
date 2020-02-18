# Thanos与Cortex方案对比

## 发展现状

Thanos项目创建于2017年12月份。在2019年8月份成为CNCF沙盒项目。目前的维护成员主要来自于Red Hat。

Cortex项目创建于2016年2月份。在2018年8月成为CNCF沙盒项目。项目维护成员主要来自于Weavework。

| 项目名称 | Watch | Star | Fork | Issues | Contributors |
| :--- | :--- | :--- | :--- | :--- | :--- |
| [Thanos](https://github.com/thanos-io/thanos) | 157 | 5.2k | 654 | 87 | 188 |
| [Cortex](https://github.com/cortexproject/cortex) | 90 | 2.4k | 296 | 230 | 90 |

从项目发展上讲Thano相对于Cortex更加年轻，在活跃度上呈后来者居上的态势。且目前Prometheus项目中已经引入了Thanos的部署用例和代码库，两者的契合度相对较高。Cortex目前实验性的blocks数据块写入后端存储的特性也复用了Thanos的代码。

## 集群架构

Cortex的定位是提供Prometheus as a Service，采用中心化的架构。Prometheus可能来自于多个数据中心，所有的数据都汇入一个Cortex数据中心，由Cortex集中写入和查询。

![](../../.gitbook/assets/image%20%2811%29.png)



Thanos采用的是分散式的架构，其数据可以分散在多个数据中心，且分别存储于多个对象存储桶中。对于数据的查询可分散也可集中。

![](../../.gitbook/assets/image%20%282%29.png)

## 特性对比

| 特性名称 | Thanos | Cortex |
| :--- | :--- | :--- |
| 全局查询 | 集群级支持 | 租户级支持 |
| 全局告警 | 集群级支持 | 租户级支持 |
| 高可用 | 部分支持 | 支持 |
| 长期存储 | 支持 | 支持 |
| 多集群 | 支持 | 不支持 |
| 多租户 | 部分支持 | 支持 |
| 集群拓展 | 支持 | 支持 |
| 查询缓存 | 支持 | 支持 |
| 查询拆分 | 按时间区间拆分 | 按天拆分 |
| 下采样 | 支持 | 不支持 |
| 数据去重 | 查询时去重 | 写入时去重 |
| 存储时效 | 支持 | 不支持 |

### 数据写入

Thanos主要使用pull方式的数据写入，通过Sidecar将Prometheus在本地生成的长期只读数据上传到对象存储中。

Cortex使用push方式的数据写入，prometheus通过remote wirte接口将数据写入到Distributor组件中，由Distributor分发给Ingester将数据写入到存储后端。

值得注意的是Thanos新推出的实验性组件Receiver也采用与Cortex相同的数据写入方式，通过Prometheus的Remote Write接口将数据写入Receiver组件，也使用哈希环的的方式保证写入数据的多副本可靠性。

### 数据存储

Thanos使用TSDB的blocks存储，每个blocks块初始存储2小时长的数据，后续在长期存储中经过压缩存储时长会增大。

Cortex使用TSDB的chunks存储，每个chunks块存储12小时长的数据。新推出的实验性特性将支持blocks存储。

Thanos将长期存储的数据都存放于对象存储中。

Cortex则将index和chunks分开存储，index存储于NoSQL键值存储中如BigTable，Cassandra和DynamoDB，chunks存储于BigTable，Cassandra，DynamoDB和对象存储中。其实验性的blocks存储只支持存放于对象存储。

### 数据去重

Thanos在读取时才对数据进行去重，而Cortex在写入时就会进行去重。这意味着Thanos会存储多份的冗余数据。存储的开销也会相应地更大。

### 查询优化

目前prometheus的查询痛点在于做大时间跨度的查询时需要耗费大量的时间和内存资源。

Thanos采取的优化方式包括对长期存储的数据进行下采样，在查询大跨度时间的数据时使用采样跨度更大的数据块做查询。除此之外还可以按时间段进行拆分，分配每个长期存储网关所负责查询的时间段。还可以安装存储桶进行拆分，数据写入到多个存储桶中，而每个长期存储网关负责查询一个存储桶。时间拆分和桶拆分可以同时使用，从部署架构层面进行优化。

Cortex采取的优化方式是将数据进行缓存，在查询时通过读取缓存中的index和chunks，并生成子查询用以补足缓存中所遗漏的内容。在查询多天的大查询时会将其拆分为多个单天的子查询并行处理。并且查询后的结果也会被缓存下来用于后续的查询。

### 多租户

Cortex的多租户实现了租户的查询和告警相关配置的隔离。所有的Cortex组件在发送的请求中都会携带一个Header X-Scope-OrgID，表示租户ID。当Ingester在写入数据时发现有一个新的租户ID时，则认为这是一个新的租户，当Querier在读取数据时，会请求中的租户ID是否已经存在。创建新租户的过程是随着指标数据的写入自动完成的，不存在租户的创建或删除接口，也不存在租户的鉴权和认证，这些需要在入口网关中实现。

Thanos的多租户目前只是实验性功能且不完整，其中的Reciever组件实现了多租户的数据写入隔离，与Cortex相似也是使用了请求头携带租户ID，相当于Cortex中Distributor和Ingester的结合体。对于多租户的查询目前推荐的方式是为每个租户的Prometheus+Sidecar设置专门查询用的Query，或者是通过[prom-label-proxy](https://github.com/openshift/prom-label-proxy)来为查询强制添加过滤标签，以标签作为租户隔离的依据。

### 多集群

Thanos的集群架构式分散式的，依靠Querier组件的分层架构支持了跨多个Thanos集群的查询和评估告警。

Cortex的集群架构是中心式的，目前暂不支持跨集群。

### 高可用

Thanos中的Compact和Rule组件都是单例部署，暂不支持高可用。

Cortex中的所有组件均实现了多副本部署，支持高可用。

### 集群拓展

Thanos集群具有灵活的拓展性，对于内部的集群可以通过Sidecar+Prometheus拓展，对于外部的集群可以通过Prometheus+Receiver拓展。

Cortex的集群拓展只需要加新的Prometheus实例对接到写入网关上即可。

## 维护成本

### 资源开销

Thanos中对于内存开销较大的组件是Store，该组件会将部分查询频繁的数据块索引进行缓存。

Cortex中对于内存开销较大的组件是缓存中间件，这其中缓存了index，chunks和查询结果。

相比之下Cortex缓存的数据会更多，占用的内存开销更大。

### 第三方组件依赖

Thanos的外部组件依赖仅有对象存储后端（存放长期数据）。而Cortex的外部组件依赖包括了Postgres（存放各租户的规则配置），Consul键值存储（集群治理，后续会以gossip替代），存储后端（存放长期数据），Redis或Memcached（查询缓存）。

更多的组件依赖使得Cortex的维护难度会更大。

### 部署配置复杂性

Thanos的部署配置目前分为两个部分：

1. 通过[prometheus-operator](https://github.com/coreos/prometheus-operator)部署Prometheus+Sidecar，包括配置每个prometheus集群所对应的recording和alerting规则；
2. 通过[helm charts](https://hub.helm.sh/charts?q=thanos)部署配置其他的组件。

Cortex的部署配置可以使用[helm charts](https://github.com/cortexproject/cortex-helm-chart)，对于多租户的告警相关配置需要通过接口请求配置。

### 其他痛点

Thanos Query会向多个StoreAPI服务发起查询请求，汇聚所有的请求结果后才会响应，随着集群架构的扩展，StoreAPI服务的数量增加导致了出现连接问题的可能性也会增加，为了避免部分查询对整体的影响，Querier上实现了部分响应的特性，根据设置条件（如超时时间）使得查询不会出现过长的等待。然而这又带来了另一个问题，查询的返回结果可能会不完整，需要根据场景在查询稳定性和完整性上做取舍。

### 参考资料

{% embed url="https://www.youtube.com/watch?time\_continue=80&v=KmJnmd3K3Ws&feature=emb\_logo" %}



