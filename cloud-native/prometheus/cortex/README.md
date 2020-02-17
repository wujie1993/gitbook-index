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

* Distributor。作为Cortex集群的数据写入入口，接收来自prometheus的remote write API写入数据。对写入的数据进行过滤和去重，将结果存入ingester。
* Ingester。用于存储短期的指标数据，并定期将数据刷写到后端存储中。同时提供了用于查询短期指标数据的查询接口。
* Querier。
* Query Frontend
* Ruler

## 工作机制

### 数据写入

### 数据读取

### 告警发送

## 优势特性

* 全局查询。
* 水平拓展。
* 长期存储。
* 多租户。

## 缺点

依赖需要多个数据库

