# 存储方案差异对比

### 数据写入

thanos主要使用sidecar上传数据到对象存储中

cortex主要使用prometheus的remote wirte接口写入数据
