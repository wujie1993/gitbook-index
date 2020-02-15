# Distributor

Distributor接收来自prometheus的remote write API写入数据。首先会检查指标数据的正确性以及是否在特定租户限制的范围内，如果不在特定租户限制范围内会使用默认租户。最后把数据分多个批次并行发给Ingester。

验证指标正确性的范围包括：

* 指标的标签名称格式是否正确。
* 指标的标签数量是否在配置的最大数量限制内。
* 指标的标签名称和值是否在配置的最大长度限制内。
* 指标的时间戳是否在配置的限制时间范围内。

Distributor服务本身是无状态的可以按需进行水平拓展。

### High Avaliablity Tracker

当开启**High Availability Tracker（HA Tracker）**特性后。Distributor会对来自冗余副本的Promtheus的指标进行去重，这使得prometheus可以配置多个副本实例以实现高可用，并且写入数据时在Distributor上进行去重。

HA Tracker根据prometheus上配置的集群标签和副本标签进行去重，其中集群标签与租户一一对应，一个集群中包含着多个prometheus冗余副本，副本标签表示一个prometheus集群中每个冗余副本的编号。当判断到写入的指标不是来自于一个集群中的主要副本时，写入的内容会被丢弃掉。

如何确定prometheus的主要副本需要依赖于键值存储数据库。

