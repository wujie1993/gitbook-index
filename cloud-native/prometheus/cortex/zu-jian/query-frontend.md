# Query Frontend

query frontend是一个可选的组件，用于加速指标的查询。在部署query frontend之后，所有对于指标的查询请求都要定向到query frontend而不是querier。

在接收到查询请求后query frontend不会立即响应，而是将查询请求放入一个查询队列中，querier会连接到query frontend并消费这个队列，执行从队列中获取的查询请求并响应给query frontend，query frontend会对这些响应的结果进行聚合。因此，在querier上需要配置query frontend的地址（通过启动参数`-querier.frontend-address`）用以连接到query frontend。

query frontend是有状态服务，本身需要存储数据，一般建议运行2个副本以确保高可用和调度的均衡。

### 查询队列

query frontend的队列机制有以下用途。

* 确保可能导致OOM的大型查询在发生错误时能够得到重试。
* 防止多个大的查询请求打在单个querier上。
* 可以分配租户所对应的querier，避免单个租户使用DOS拒绝服务攻击其他租户。

### 查询拆分

query frontend会将多天的的查询拆分为多个单天的查询，游下游的querier去并行处理这些已拆分的查询。返回的查询结果由query frontend进行汇聚。这样可以防止大时间跨度的查询导致queier发生OOM，并且能够更快的执行查询。

### 查询缓存

query frontend支持将查询结果进行缓存用以加速后续的查询。当缓存的结果不够完整时，query frontend会计算出所需要的子查询并分配给下游的querier并行执行。子查询的步长会对齐以提升查询结果的可缓存性。当前支持的缓存后端有：memcached，redis和内存。

