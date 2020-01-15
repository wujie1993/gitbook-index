# 整体结构

## 架构图

![](../../.gitbook/assets/image%20%286%29.png)

## 组件介绍

thanos包括以下组件：

* Sidecar。主要用于短期数据的查询和数据上传。以gRPC的方式暴露StoreAPI接口，在Querier查询时将请求转为PromSQL代理到Prometheus并返回Querier。同时实现了本地文件的嗅探器，将Promethues所产生的本地数据上传到远端对象存储中。
* Store Gateway。主要用于长期数据的查询。以gRPC的方式暴露StoreAPI接口，在Querier查询时读取远端对象存储中的数据并返回Querier。
* Compactor。定期地读取远端对象存储中的数据，进行压缩和降低精度后保存回对象存储中。
* Receiver。接收Prometheus的远程写入数据并上传到云存储中
* Ruler/Rule。通过Querier的Prometheus查询接口定期地获取指标并评估record和alert规则，将record规则的评估结果保存到本地，通过嗅探器将文件上传到远端对象存储中，将alert规则的评估结果用于触发Alertmanager告警。
* Querier/Query。作为指标查询入口，实现了Prometheus的查询接口，在接收到查询请求后通过StoreAPI转发请求到Sidecar和Store Gateway，将结果进行汇聚去重后返回客户端，同时自身也实现了StoreAPI，可处理来自于其他Querier的查询请求。Querier本身集成了与Prometheus类似的UI面板。



