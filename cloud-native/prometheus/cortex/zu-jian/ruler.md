# Ruler

Ruler是可选的安装组件，用于评估recording和alerting规则。需要依靠数据库存储recording规则，以及为对应的租户发送告警消息到alertmanager上。

### 配置读取

### recording规则

ruler从配置数据库中读取到recording规则，根据规则生成对应的查询请求发送到ingester获取到评估所需的指标，评估后的最终结果最终写入到ingester上。

### alerting规则

ruler从配置数据库中读取到recording规则，根据规则生成对应的查询请求发送到ingester获取评估所需的指标，当告警条件成立时，会发送告警到alertmanager上。

