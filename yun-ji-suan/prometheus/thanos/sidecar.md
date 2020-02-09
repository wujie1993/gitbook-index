# Sidecar

Sidecar主要用于短期数据的查询和Prometheus本地数据上传

每个Prometheus实例都会对应运行着一个Sidecar，其本体主要分为两个部分：StoreAPI和Snipper

StoreAPI是Thanos内部约定的一套gRPC查询接口，对Prometheus原生的查询接口做了二次封装。外部（如：Querier）在查询指标时会将请求发送到Sidecar的StoreAPI，由Sidecar将该请求转为PromSQL代理到自身所绑定的Prometheus查询接口上，将Prometheus返回的结果通过StoreAPI返回给查询方。

Snipper用于本地文件的嗅探和上传。当Prometheus在本地生成新的只读块（非Head Block）时，会将该数据块上传到远端对象存储中。做为长期历史数据。

### 远端存储

sidecar可以将prometheus产生的数据块上传到远端对象存储中，做为历史数据长期保存，后续可以通过compactor对远端的数据进行压缩，querier也可以通过store gateway读取到长期的历史数据

为sidecar设定启动参数`--objstore.*`开启远端存储

在开启远端存储的同时，需要将prometheus的数据压缩关闭，将启动参数`--storage.tsdb.min-block-duration`和`--storage.tsdb.max-block-duration`设置为相同值。

retention建议不小于block-duration的三倍

例子：开启远端存储

{% tabs %}
{% tab title="prometheus启动参数" %}
```text
prometheus \
  --storage.tsdb.max-block-duration=2h \
  --storage.tsdb.min-block-duration=2h \
  --storage.tsdb.retention.time=6h \
  --web.enable-lifecycle
```
{% endtab %}

{% tab title="sidecar启动参数" %}
```
thanos sidecar \
  --tsdb.path        "/path/to/prometheus/data/dir" \
  --prometheus.url   "http://localhost:9090" \
  --objstore.config-file  "bucket.yml"
```
{% endtab %}

{% tab title="bucket.yml对象存储配置" %}
```
type: S3
config:
  bucket: "thanos"
  endpoint: "minio-my-store.rook-minio:9000"
  region: "default"
  access_key: "TEMP_DEMO_ACCESS_KEY"
  insecure: true
  signature_version2: true
  encrypt_sse: false
  secret_key: "TEMP_DEMO_SECRET_KEY"
  put_user_metadata: {}
  http_config:
    idle_conn_timeout: 90s
    response_header_timeout: 2m
    insecure_skip_verify: true
  trace:
    enable: true
  part_size: 134217728   
```
{% endtab %}
{% endtabs %}

prometheus-operator也支持thanos的远端存储配置，通过参数`.spec.thanos.objectStorageConfig`指定对象存储配置，通过参数`.spec.disableCompaction: true`禁用prometheus的块压缩

例子：通过prometheus-operator开启远端存储

{% tabs %}
{% tab title="prometheus+sidecar CR定义" %}
```text
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: self
  namespace: thanos
  labels:
    prometheus: self
spec:
  replicas: 2
  serviceMonitorSelector:
    matchLabels:
      app: prometheus
  ruleSelector:
    matchLabels:
      role: prometheus-rulefiles
      prometheus: k8s
  thanos:
    version: v0.5.0
    objectStorageConfig:
      name: objectstore
      key: config
      optional: true
  disableCompaction: true
```
{% endtab %}

{% tab title="对象存储配置" %}
```
kind: Secret
apiVersion: v1
metadata:
  name: objectstore
  namespace: thanos
data:
  config: >-
    dHlwZTogUzMKY29uZmlnOgogIGJ1Y2tldDogInRoYW5vcyIKICBlbmRwb2ludDogIm1pbmlvLW15LXN0b3JlLnJvb2stbWluaW86OTAwMCIKICByZWdpb246ICJkZWZhdWx0IgogIGFjY2Vzc19rZXk6ICJURU1QX0RFTU9fQUNDRVNTX0tFWSIKICBpbnNlY3VyZTogdHJ1ZQogIHNpZ25hdHVyZV92ZXJzaW9uMjogdHJ1ZQogIGVuY3J5cHRfc3NlOiBmYWxzZQogIHNlY3JldF9rZXk6ICJURU1QX0RFTU9fU0VDUkVUX0tFWSIKICBwdXRfdXNlcl9tZXRhZGF0YToge30KICBodHRwX2NvbmZpZzoKICAgIGlkbGVfY29ubl90aW1lb3V0OiA5MHMKICAgIHJlc3BvbnNlX2hlYWRlcl90aW1lb3V0OiAybQogICAgaW5zZWN1cmVfc2tpcF92ZXJpZnk6IHRydWUKICB0cmFjZToKICAgIGVuYWJsZTogdHJ1ZQogIHBhcnRfc2l6ZTogMTM0MjE3NzI4ICAgCg==
type: Opaque
```
{% endtab %}
{% endtabs %}



