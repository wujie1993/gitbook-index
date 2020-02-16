# Distributor

Distributor接收来自prometheus的remote write API写入数据。首先会检查指标数据的正确性以及是否在特定租户限制的范围内，如果不在特定租户限制范围内会使用默认租户。最后把数据分多个批次并行发给Ingester。

验证指标正确性的范围包括：

* 指标的标签名称格式是否正确。
* 指标的标签数量是否在配置的最大数量限制内。
* 指标的标签名称和值是否在配置的最大长度限制内。
* 指标的时间戳是否在配置的限制时间范围内。

Distributor服务本身是无状态的可以按需进行水平拓展。

### 高可用追踪器

当开启**High Availability Tracker（HA Tracker）**特性后。Distributor会对来自冗余副本的Promtheus的指标进行去重，这使得prometheus可以配置多个副本实例以实现高可用，并且写入数据时在Distributor上进行去重。

HA Tracker根据prometheus上配置的集群标签和副本标签进行去重，其中集群标签与租户一一对应，一个集群中包含着多个prometheus冗余副本，副本标签表示一个prometheus集群中每个冗余副本的编号。当判断到写入的指标不是来自于一个集群中的主要副本时，写入的内容会被丢弃掉。

如何协调prometheus的主要副本需要依赖于键值存储数据库。Distributor只会接收来自于与主要副本的写入数据。当写入的指标仅携带了集群和副本标签中的一个或者不携带时，Distributor会默认接收并且不做任何去重处理。

目前支持的键值存储数据库有：Consul和Etcd。

### 哈希计算

Distributor通过一致性哈希判断写入指标的对应到哪个ingester实例上，有两种哈希计算策略：

* 将指标名称和租户ID计算哈希（默认）
* 将指标名称，指标标签和租户ID计算哈希（需要启用参数`-distributor.shard-by-all-labels=true`）

这种方式使得写入数据可以在多个ingestor上达到均衡，而在读取的时候querier需要与所有ingester通讯因为不同标签的指标会分布在不同的ingestor上。

每个ingestor在启动后会生成一个随机的无符号32整型值作为自身的token，并在键值存储中进行服务自注册。所有的token会根据大小从小打到大排序并首尾相连形成一个闭环，称为哈希环。每个ingestor会以自身token值到前一个token值的这段区间作为自身所负责的哈希范围。Distributor将指标写入到对应的ingestor实例上。

例子: 有三个ingester实例，token值分别为0，25，50。哈希值为1-25的指标会落在token值为25的ingester上，哈希值为26-50的指标会落在token值为50的ingester上，其余的会落在token值为0的ingester上。

目前哈希环所支持的键值存储数据库有：console，etcd和gossip memberlist（实验性）。

### 仲裁一致性

Cortex使用[Dynamo-style](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)确保读和写的仲裁一致性，Distributor只会在超过半数需要写入数据的ingester完成写入响应后才会响应写入成功到prometheus。

