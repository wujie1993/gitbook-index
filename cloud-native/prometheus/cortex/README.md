# Cortex

## 项目简介

Cortex是基于Proemtheus实现的集群监控方案，具有水平拓展，高可用，多租户，长期存储等特点。

项目由Tom Wilkie和Julius Volz创建于2016年2月份。在2018年8月成为CNCF沙盒项目。项目维护成员主要来自于Weavework。

## 官方网站

{% embed url="https://cortexmetrics.io/" %}

## 项目地址

{% embed url="https://github.com/cortexproject/cortex" %}

## 架构图

![](https://cortexmetrics.io/images/architecture.png)

## 组件介绍

* Config。作为租户的alertmanager配置，规则和模板的配置接口。
* Distributor。作为Cortex集群的数据写入入口，接收来自prometheus的remote write API写入数据。对写入的数据进行过滤和去重，将结果存入ingester。
* Ingester。用于存储短期的指标数据，并定期将数据刷写到后端存储中。同时提供了用于查询短期指标数据的查询接口。
* Querier。实现了与prometheus相同的查询接口，处理来自外部的查询请求。对TSDB的index和chunks进行缓存以加速查询。在部署了Query Frontend的情况下则作为处理子查询的工作节点。
* Query Frontend。实现了与prometheus相同的查询接口，处理来自外部的查询请求。将单个大查询拆分为多个子查询并放入查询队列，交由querier完成查询后进行汇聚，同时也会对查询结果进行缓存用以加速查询。
* Ruler。用以评估recording和alerting规则。将recording结果写入ingester，将告警消息发送到alertmanager。

## 工作机制

### 数据写入

### 数据读取

### 告警发送

## 优势特性

* 全局查询。
* 水平拓展。
* 长期存储。Ingester做为长期数据写入的唯一入口，每当本地生成12小时长的chunks时，会将该chunks以及对应的index上传到存储后端，需要注意的是index和chunks分开存储。
* 多租户。所有的Cortex组件在发送的请求中都会携带一个Header X-Scope-OrgID，表示租户ID。当Ingester在写入数据时发现有一个新的租户ID时，则认为这是一个新的租户，当Querier在读取数据时，会请求中的租户ID是否已经存在。创建新租户的过程是随着指标数据的写入自动完成的，不存在租户的创建或删除接口，也不存在租户的鉴权和认证，这些需要在入口网关中实现。

## 缺点

1. 需要依赖多种第三方中间件，提升了运维的难度
2. 官方文档中部分组件的介绍不够完整甚至没有介绍

