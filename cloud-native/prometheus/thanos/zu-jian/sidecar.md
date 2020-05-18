# Sidecar

Sidecar主要用于短期数据的查询和Prometheus本地数据上传。

每个Prometheus实例都会对应运行着一个Sidecar，其本体主要分为两个部分：StoreAPI和Snipper

StoreAPI是Thanos内部约定的一套gRPC查询接口，对Prometheus原生的查询接口做了二次封装。外部（如：Querier）在查询指标时会将请求发送到Sidecar的StoreAPI，由Sidecar将该请求转为PromSQL代理到自身所绑定的Prometheus查询接口上，将Prometheus返回的结果通过StoreAPI返回给查询方。

Shipper用于本地文件的嗅探和上传。当Prometheus在本地生成新的只读块（非Head Block）时，会将该数据块上传到远端对象存储中，做为长期历史数据。在上传数据块时，会为数据块中的meta.json添加thanos字段，并在其中附加prometheus实例的外部标签。在meta.json中添加的外部标签可用于方便后续做查询时在不访问对象存储数据的情况下过滤数据块。

### 可选参数

```text
usage: thanos sidecar [<flags>]

sidecar for Prometheus server

Flags:
  -h, --help                     Show context-sensitive help (also try
                                 --help-long and --help-man).
      --version                  Show application version.
      --log.level=info           Log filtering level.
      --log.format=logfmt        Log format to use. Possible options: logfmt or
                                 json.
      --tracing.config-file=<file-path>
                                 Path to YAML file with tracing configuration.
                                 See format details:
                                 https://thanos.io/tracing.md/#configuration
      --tracing.config=<content>
                                 Alternative to 'tracing.config-file' flag
                                 (lower priority). Content of YAML file with
                                 tracing configuration. See format details:
                                 https://thanos.io/tracing.md/#configuration
      --http-address="0.0.0.0:10902"
                                 Listen host:port for HTTP endpoints.
      --http-grace-period=2m     Time to wait after an interrupt received for
                                 HTTP Server.
      --grpc-address="0.0.0.0:10901"
                                 Listen ip:port address for gRPC endpoints
                                 (StoreAPI). Make sure this address is routable
                                 from other components.
      --grpc-grace-period=2m     Time to wait after an interrupt received for
                                 GRPC Server.
      --grpc-server-tls-cert=""  TLS Certificate for gRPC server, leave blank to
                                 disable TLS
      --grpc-server-tls-key=""   TLS Key for the gRPC server, leave blank to
                                 disable TLS
      --grpc-server-tls-client-ca=""
                                 TLS CA to verify clients against. If no client
                                 CA is specified, there is no client
                                 verification on server side. (tls.NoClientCert)
      --prometheus.url=http://localhost:9090
                                 URL at which to reach Prometheus's API. For
                                 better performance use local network.
      --prometheus.ready_timeout=10m
                                 Maximum time to wait for the Prometheus
                                 instance to start up
      --receive.connection-pool-size=RECEIVE.CONNECTION-POOL-SIZE
                                 Controls the http MaxIdleConns. Default is 0,
                                 which is unlimited
      --receive.connection-pool-size-per-host=100
                                 Controls the http MaxIdleConnsPerHost
      --tsdb.path="./data"       Data directory of TSDB.
      --reloader.config-file=""  Config file watched by the reloader.
      --reloader.config-envsubst-file=""
                                 Output file for environment variable
                                 substituted config file.
      --reloader.rule-dir=RELOADER.RULE-DIR ...
                                 Rule directories for the reloader to refresh
                                 (repeated field).
      --objstore.config-file=<file-path>
                                 Path to YAML file that contains object store
                                 configuration. See format details:
                                 https://thanos.io/storage.md/#configuration
      --objstore.config=<content>
                                 Alternative to 'objstore.config-file' flag
                                 (lower priority). Content of YAML file that
                                 contains object store configuration. See format
                                 details:
                                 https://thanos.io/storage.md/#configuration
      --shipper.upload-compacted
                                 If true sidecar will try to upload compacted
                                 blocks as well. Useful for migration purposes.
                                 Works only if compaction is disabled on
                                 Prometheus. Do it once and then disable the
                                 flag when done.
      --min-time=0000-01-01T00:00:00Z
                                 Start of time range limit to serve. Thanos
                                 sidecar will serve only metrics, which happened
                                 later than this value. Option can be a constant
                                 time in RFC3339 format or time duration
                                 relative to current time, such as -1d or 2h45m.
                                 Valid duration units are ms, s, m, h, d, w, y.
```

### 远端存储

sidecar可以将prometheus所产生的未压缩过的数据块上传到远端对象存储中，做为历史数据长期保存，后续可以通过compactor对远端的数据进行压缩，querier也可以通过store读取到长期的历史数据。

为sidecar设定启动参数`--objstore.*`开启远端存储。在开启远端存储的同时，需要将prometheus的数据压缩关闭，将启动参数`--storage.tsdb.min-block-duration`和`--storage.tsdb.max-block-duration`设置为相同值。

`--store.tsdb.retention.time`表示本地存储数据的最大时长，建议不小于block-duration\(默认2小时\)的三倍。

例子：开启远端存储

{% tabs %}
{% tab title="prometheus 启动参数" %}
```text
prometheus \
  --storage.tsdb.max-block-duration=2h \
  --storage.tsdb.min-block-duration=2h \
  --storage.tsdb.retention.time=6h \
  --web.enable-lifecycle
```
{% endtab %}

{% tab title="sidecar 启动参数" %}
```
thanos sidecar \
  --tsdb.path        "/path/to/prometheus/data/dir" \
  --prometheus.url   "http://localhost:9090" \
  --objstore.config-file  "bucket.yml"
```
{% endtab %}

{% tab title="bucket.yml 对象存储配置" %}
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

{% tab title="objectstore 对象存储配置" %}
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

### 上传压缩块

当需要从原生的prometheus中将数据迁移到thanos中使用时，可以在启动时附加`--shipper.upload-compacted`参数。该参数开启后sidecar会判断本地数据块中meta.json的compaction.level值，大于1的表示该数据块已经被压缩过，被压缩过的块也会被上传到对象存储中做为历史数据保存。该功能目前为实验性功能，在开启时需要关闭prometheus的压缩功能。

