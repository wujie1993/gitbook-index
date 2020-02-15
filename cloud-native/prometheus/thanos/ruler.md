# Ruler

rule组件用于评估prometheus的recording和alerting规则。其本身不会抓取metrics接口数据，而是通过query API从query组件定期地获取指标数据，如果配置了多个query地址，则会采用轮询方式获取。

其中recording规则评估生成地数据会以prometheus 2.0格式保存在本地，并且定期地扫描本地生成的TSDB数据块上传到对象存储桶中做为历史数据长期保存。同时也实现了StoreAPI可用于查询本地保存的数据。

与prometheus节点类似，每个rule节点都使用独立的存储，可以同时运行多个副本，而且需要为每个副本实例分配不同的标签以作区分，因为store组件在查询对象存储中的历史数据时是以该标签进行分组查询。

### 可用参数

```text
usage: thanos rule [<flags>]

ruler evaluating Prometheus rules against given Query nodes, exposing Store API and storing old blocks in bucket

Flags:
  -h, --help                     Show context-sensitive help (also try --help-long and --help-man).
      --version                  Show application version.
      --log.level=info           Log filtering level.
      --log.format=logfmt        Log format to use.
      --tracing.config-file=<file-path>
                                 Path to YAML file with tracing configuration. See format details: https://thanos.io/tracing.md/#configuration
      --tracing.config=<content>
                                 Alternative to 'tracing.config-file' flag (lower priority). Content of YAML file with tracing configuration. See format details:
                                 https://thanos.io/tracing.md/#configuration
      --http-address="0.0.0.0:10902"
                                 Listen host:port for HTTP endpoints.
      --http-grace-period=2m     Time to wait after an interrupt received for HTTP Server.
      --grpc-address="0.0.0.0:10901"
                                 Listen ip:port address for gRPC endpoints (StoreAPI). Make sure this address is routable from other components.
      --grpc-grace-period=2m     Time to wait after an interrupt received for GRPC Server.
      --grpc-server-tls-cert=""  TLS Certificate for gRPC server, leave blank to disable TLS
      --grpc-server-tls-key=""   TLS Key for the gRPC server, leave blank to disable TLS
      --grpc-server-tls-client-ca=""
                                 TLS CA to verify clients against. If no client CA is specified, there is no client verification on server side. (tls.NoClientCert)
      --label=<name>="<value>" ...
                                 Labels to be applied to all generated metrics (repeated). Similar to external labels for Prometheus, used to identify ruler and its blocks as unique source.
      --data-dir="data/"         data directory
      --rule-file=rules/ ...     Rule files that should be used by rule manager. Can be in glob format (repeated).
      --resend-delay=1m          Minimum amount of time to wait before resending an alert to Alertmanager.
      --eval-interval=30s        The default evaluation interval to use.
      --tsdb.block-duration=2h   Block duration for TSDB block.
      --tsdb.retention=48h       Block retention time on local disk.
      --alertmanagers.url=ALERTMANAGERS.URL ...
                                 Alertmanager replica URLs to push firing alerts. Ruler claims success if push to at least one alertmanager from discovered succeeds. The scheme should not be empty e.g
                                 `http` might be used. The scheme may be prefixed with 'dns+' or 'dnssrv+' to detect Alertmanager IPs through respective DNS lookups. The port defaults to 9093 or the
                                 SRV record's value. The URL path is used as a prefix for the regular Alertmanager API path.
      --alertmanagers.send-timeout=10s
                                 Timeout for sending alerts to Alertmanager
      --alertmanagers.config-file=<file-path>
                                 Path to YAML file that contains alerting configuration. See format details: https://thanos.io/components/rule.md/#configuration. If defined, it takes precedence over
                                 the '--alertmanagers.url' and '--alertmanagers.send-timeout' flags.
      --alertmanagers.config=<content>
                                 Alternative to 'alertmanagers.config-file' flag (lower priority). Content of YAML file that contains alerting configuration. See format details:
                                 https://thanos.io/components/rule.md/#configuration. If defined, it takes precedence over the '--alertmanagers.url' and '--alertmanagers.send-timeout' flags.
      --alertmanagers.sd-dns-interval=30s
                                 Interval between DNS resolutions of Alertmanager hosts.
      --alert.query-url=ALERT.QUERY-URL
                                 The external Thanos Query URL that would be set in all alerts 'Source' field
      --alert.label-drop=ALERT.LABEL-DROP ...
                                 Labels by name to drop before sending to alertmanager. This allows alert to be deduplicated on replica label (repeated). Similar Prometheus alert relabelling
      --web.route-prefix=""      Prefix for API and UI endpoints. This allows thanos UI to be served on a sub-path. This option is analogous to --web.route-prefix of Promethus.
      --web.external-prefix=""   Static prefix for all HTML links and redirect URLs in the UI query web interface. Actual endpoints are still served on / or the web.route-prefix. This allows thanos UI
                                 to be served behind a reverse proxy that strips a URL sub-path.
      --web.prefix-header=""     Name of HTTP request header used for dynamic prefixing of UI links and redirects. This option is ignored if web.external-prefix argument is set. Security risk: enable
                                 this option only if a reverse proxy in front of thanos is resetting the header. The --web.prefix-header=X-Forwarded-Prefix option can be useful, for example, if Thanos
                                 UI is served via Traefik reverse proxy with PathPrefixStrip option enabled, which sends the stripped prefix value in X-Forwarded-Prefix header. This allows thanos UI
                                 to be served on a sub-path.
      --objstore.config-file=<file-path>
                                 Path to YAML file that contains object store configuration. See format details: https://thanos.io/storage.md/#configuration
      --objstore.config=<content>
                                 Alternative to 'objstore.config-file' flag (lower priority). Content of YAML file that contains object store configuration. See format details:
                                 https://thanos.io/storage.md/#configuration
      --query=<query> ...        Addresses of statically configured query API servers (repeatable). The scheme may be prefixed with 'dns+' or 'dnssrv+' to detect query API servers through respective
                                 DNS lookups.
      --query.sd-files=<path> ...
                                 Path to file that contain addresses of query peers. The path can be a glob pattern (repeatable).
      --query.sd-interval=5m     Refresh interval to re-read file SD files. (used as a fallback)
      --query.sd-dns-interval=30s
                                 Interval between DNS resolutions.
```

规则文件的配置格式与prometheus相同，其中`interval`参数默认与启动参数`--eval-interval`一致，也可单独指定，每个规则评估任务会以该时间做为周期向query组件发送查询请求获取评估所需的指标数据。

### 部分响应

rule对原有的prometheus规则配置做了拓展，在每条规则配置上新加了字段partial\_response\_strategy用以指定向query发起查询时的部分响应策略。可选值有`warn`和`abort`（默认值）。

warn表示忽略所有的StoreAPI响应异常并返回结果，在控制台打印警告日志。

abort表示当有StoreAPI响应异常时直接终止查询并在控制台打印错误日志。

例子：

```text
groups:
- name: "warn strategy"
  partial_response_strategy: "warn"
  rules:
  - alert: "some"
    expr: "up"
- name: "abort strategy"
  partial_response_strategy: "abort"
  rules:
  - alert: "some"
    expr: "up"
- name: "by default strategy is abort"
  rules:
  - alert: "some"
    expr: "up"
```

### 高可用

rule组件支持高可用部署，每个副本实例都会使用独立的存储空间和特定的外部标签，为了避免发送到alertmanager的告警产生重复告警，需要在启动时附加参数`--alert.label-drop="{{ 外部标签键 }}"`以消除告警通知中的不同标签。alertmanager在判断告警通知时判断标签是相同的会认定为是同一个告警，不会重复发送。

### 告警对接

可通过启动参数 `--alertmanagers.config` 或`--alertmanagers.config-file`指定对接alertmanager的配置，格式如下：

```text
alertmanagers:
- http_config:
    basic_auth:
      username: ""
      password: ""
      password_file: ""
    bearer_token: ""
    bearer_token_file: ""
    proxy_url: ""
    tls_config:
      ca_file: ""
      cert_file: ""
      key_file: ""
      server_name: ""
      insecure_skip_verify: false
  static_configs: []
  file_sd_configs:
  - files: []
    refresh_interval: 0s
  scheme: http
  path_prefix: ""
  timeout: 10s
  api_version: v1
```

其中`api_version`支持`v1`和`v2`。可以配置多个alermanager接入地址，rule将所有接入地址认定为一个接收组，按配置顺序向其中的一个发起通知，只有在所有接入地址都报错的情况下才会认定为告警通知发送失败。

### 查询对接

可通过启动参数 `--query.config` and `--query.config-file`指定对接query的配置，格式如下：

```text
- http_config:
    basic_auth:
      username: ""
      password: ""
      password_file: ""
    bearer_token: ""
    bearer_token_file: ""
    proxy_url: ""
    tls_config:
      ca_file: ""
      cert_file: ""
      key_file: ""
      server_name: ""
      insecure_skip_verify: false
  static_configs: []
  file_sd_configs:
  - files: []
    refresh_interval: 0s
  scheme: http
  path_prefix: ""
```

与告警对接相同，query的接入地址也可配置多个，rule按配置的顺序向其中的一个发起评估查询，只有所有query都响应失败的情况下才会认定为评估查询发送失败。

{% hint style="info" %}
rule组件获取评估数据的路径 rule --&gt; query --&gt; sidecar --&gt; prometheus 需要经过一整个查询链条，这也提升了发生故障的风险，且评估原本就可以在prometheus中进行，因此，在非必要的情况下更加推荐使用prometheus做alerting和recording评估。
{% endhint %}

