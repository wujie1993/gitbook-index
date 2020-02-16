# Thanos与Cortex方案对比

### 项目情况

Thanos项目创建于2017年12月份。在2019年8月份成为CNCF沙盒项目。目前的维护成员主要来自于Red Hat。

Cortex项目创建于2016年2月份。在2018年8月成为CNCF沙盒项目。项目维护成员主要来自于Weavework。

Thanos和Cortex都具有的特性：全局查询，高可用，长期存储。

从项目发展上讲Thano相对于Cortex更加年轻，且目前Prometheus项目中已经引入了Thanos的部署用例和代码库，两者的契合度相对较高。

### 集群架构

Cortex的定位是提供Prometheus as a Service，采用中心化的架构。Prometheus可能来自于多个数据中心，所有的数据都汇入Cortex一个中心，由Cortex集中写入和查询。

Thanos采用的是分散式的架构，其数据可以分散在多个数据中心，且分别存储于多个对象存储桶中。对于数据的查询可分散也可集中。

### 数据写入

Thanos主要侧重于通过Sdecar将Prometheus在本地生成的长期只读数据上传到对象存储中。值得注意的是Thanos新推出的实验性组件Receiver也采用与Cortex相同的数据写入方式，通过Prometheus的Remote Write接口将数据写入Receiver组件，也使用哈希环的的方式保证写入数据的多副本可靠性。

Cortex主要使用prometheus的remote wirte接口将数据写入到Distributor组件中，由Distributor分发给Ingester将数据写入到存储后端。

Thanos只支持S3一种存储后端，而Cortex支持Google Cloud Tables，S3

### 参考资料

{% embed url="https://www.youtube.com/watch?time\_continue=80&v=KmJnmd3K3Ws&feature=emb\_logo" %}



