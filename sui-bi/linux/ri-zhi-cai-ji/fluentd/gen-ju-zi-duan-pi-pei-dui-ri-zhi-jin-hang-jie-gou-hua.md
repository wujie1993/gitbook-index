# 根据字段匹配对日志进行结构化

## 问题描述

有一个k8s集群，上面部署了多个应用，每个应用都有着特定的标签。在fluentd采集应用日志的时候可以拿到每个应用的标签。对于不同的应用，需要做不同的正则匹配规则进行结构化。目标是在进行日志结构化的时候，根据标签进行匹配，这样可以避免因为对每一条日志都进行匹配而导致的性能损失。

## 解决方案

### 方案1：单fluentd实例内进行rewrite tag

```text
<source>
  @type tail
  path /var/log/test/access.log
  pos_file /var/log/test/access.log.pos
  tag kubernetes.var.log.containers.**
  format none
</source>
<match kubernetes.var.log.containers.**>
  @type rewrite_tag_filter
  <rule>
    key kubernetes.labels.javaBoot
    pattern .+
    tag java.${tag}
  </rule>
</match>
<filter java.**>
  @type parser
  format /^(?<time>\d{4}-\d{1,2}-\d{1,2} \d{2}:\d{2}:\d{2}\.\d{3})\s+(?<level>[^\s]*)\s+\[(?<service>[^,]*),(?<trace>[^,]*),(?<span>[^,]*),(?<exportable>[^\]]*)\]\s+(?<pid>[^\s]*)\s+---\s+\[(?<thread>[^\]]*)\]\s+(?<class>[^\s]*)\s+:\s+(?<message>.*)/
  time_format "%Y-%m-%d %H:%M:%S\.%L"
  keep_time_key true
  key_name log
  emit_invalid_record_to_error true
</filter>
<match java.**>
  @type stdout
</match>
<match **>
  @type elasticsearch
</match>

<label @ERROR>
  <match **>
    @type stdout
  </match>      
</label>

```

### 方案2：单fluentd实例内进行relabel

### 方案3：多个fluentd实例各自通过grep过滤

