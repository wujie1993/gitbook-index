# Querier

Querier实现了与Prometheus相同的查询接口，作为指标查询的访问入口。当PromQL查询请求到达时，  
Querier会将该请求转为StoreAPI请求发送到Sidecar，Store Gateway，Ruler或其他的Querier，将所有结果进行合并去重后再返回给客户端。

![](../../../.gitbook/assets/image%20%286%29.png)

Querier本身是无状态的，可以水平拓展成多个副本以提高服务的可用性和降低单个实例的查询压力。

### 合并去重

当多个prometheus实例恰好采集了相同的数据源时，对于一个指标项的查询会得到多份结果，因为这些指标实际上标签并不相同，会携带着每个prometheus实例所特有的标签（如：region=east,replica=0）。争对这种情况Querier实现了数据去重的功能，在启动时可添加多个`--query.replica-label {{标签名}}`参数，Querier通过StoreAPI获取到查询结果后，会忽略以上的集群标签，将指标名称与指标标签完全一致的多组指标合并为一组指标，并移除集群标签，实现对于多组重复指标的合并去重。

去重可在做查询时通过附加参数指定，如：replicaLabels=region&replicaLabels=replica&dedup=1

### 部分响应

Querier在查询时会遍历每个StoreAPI数据源，将所有结果进行合并，而如果出现部分数据源无响应的情况，会导致此处查询失败。如果相对于准确性而言，可靠性更加重要，可以为Querier添加启动参数`--partial_response`，在部分数据源响应失败时，会在日志中打印警告信息，并将该错误累加到metrics接口的计数器中，忽略错误的数据源，将现有的查询结果响应给客户端。

部分响应可在做查询时通过附加参数指定，如：partial\_response=1

### 自动下采样

在做大时间范围的查询时，可以开启自动下采样，为Querier添加启动参数`--auto-downsampling`，可选值为0，5m和1h，默认值0表示不开启自动下采样。

