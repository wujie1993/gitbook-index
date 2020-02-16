# Querier

Querier实现了与Prometheus相同的查询接口，作为指标查询的访问入口。当PromQL查询请求到达时，  
Querier会将该请求转为StoreAPI请求发送到Sidecar，Store Gateway，Ruler或其他的Querier，将所有结果进行合并去重后再返回给客户端。

![](../../../../.gitbook/assets/image%20%286%29.png)

Querier本身是无状态的，可以水平拓展成多个副本以提高服务的可用性和降低单个实例的查询压力。

### 可选参数

```text
usage: thanos query [<flags>]

query node exposing PromQL enabled Query API with data retrieved from multiple
store nodes

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
      --grpc-client-tls-secure   Use TLS when talking to the gRPC server
      --grpc-client-tls-cert=""  TLS Certificates to use to identify this client
                                 to the server
      --grpc-client-tls-key=""   TLS Key for the client's certificate
      --grpc-client-tls-ca=""    TLS CA Certificates to use to verify gRPC
                                 servers
      --grpc-client-server-name=""
                                 Server name to verify the hostname on the
                                 returned gRPC certificates. See
                                 https://tools.ietf.org/html/rfc4366#section-3.1
      --web.route-prefix=""      Prefix for API and UI endpoints. This allows
                                 thanos UI to be served on a sub-path. This
                                 option is analogous to --web.route-prefix of
                                 Promethus.
      --web.external-prefix=""   Static prefix for all HTML links and redirect
                                 URLs in the UI query web interface. Actual
                                 endpoints are still served on / or the
                                 web.route-prefix. This allows thanos UI to be
                                 served behind a reverse proxy that strips a URL
                                 sub-path.
      --web.prefix-header=""     Name of HTTP request header used for dynamic
                                 prefixing of UI links and redirects. This
                                 option is ignored if web.external-prefix
                                 argument is set. Security risk: enable this
                                 option only if a reverse proxy in front of
                                 thanos is resetting the header. The
                                 --web.prefix-header=X-Forwarded-Prefix option
                                 can be useful, for example, if Thanos UI is
                                 served via Traefik reverse proxy with
                                 PathPrefixStrip option enabled, which sends the
                                 stripped prefix value in X-Forwarded-Prefix
                                 header. This allows thanos UI to be served on a
                                 sub-path.
      --query.timeout=2m         Maximum time to process query by query node.
      --query.max-concurrent=20  Maximum number of queries processed
                                 concurrently by query node.
      --query.replica-label=QUERY.REPLICA-LABEL ...
                                 Labels to treat as a replica indicator along
                                 which data is deduplicated. Still you will be
                                 able to query without deduplication using
                                 'dedup=false' parameter.
      --selector-label=<name>="<value>" ...
                                 Query selector labels that will be exposed in
                                 info endpoint (repeated).
      --store=<store> ...        Addresses of statically configured store API
                                 servers (repeatable). The scheme may be
                                 prefixed with 'dns+' or 'dnssrv+' to detect
                                 store API servers through respective DNS
                                 lookups.
      --store.sd-files=<path> ...
                                 Path to files that contain addresses of store
                                 API servers. The path can be a glob pattern
                                 (repeatable).
      --store.sd-interval=5m     Refresh interval to re-read file SD files. It
                                 is used as a resync fallback.
      --store.sd-dns-interval=30s
                                 Interval between DNS resolutions.
      --store.unhealthy-timeout=5m
                                 Timeout before an unhealthy store is cleaned
                                 from the store UI page.
      --query.auto-downsampling  Enable automatic adjustment (step / 5) to what
                                 source of data should be used in store gateways
                                 if no max_source_resolution param is specified.
      --query.partial-response   Enable partial response for queries if no
                                 partial_response param is specified.
                                 --no-query.partial-response for disabling.
      --query.default-evaluation-interval=1m
                                 Set default evaluation interval for sub
                                 queries.
      --store.response-timeout=0ms
                                 If a Store doesn't send any data in this
                                 specified duration then a Store will be ignored
                                 and partial data will be returned if it's
                                 enabled. 0 disables timeout.
```

### 合并去重

当多个prometheus实例恰好采集了相同的数据源时，对于一个指标项的查询会得到多份结果，因为这些指标实际上标签并不相同，会携带着每个prometheus实例所特有的标签（如：region=east,replica=0）。

针对这种情况Querier实现了数据去重的功能，在启动时可添加多个`--query.replica-label {{标签名}}`参数。Querier通过StoreAPI获取到查询结果后，会移除集群标签，将指标名称与指标标签完全一致的多组指标按照时间顺序合并为一组指标。

去重可在做查询时通过附加参数指定，如：replicaLabels=region&replicaLabels=replica&dedup=1

### 部分响应

Querier在查询时会遍历每个StoreAPI数据源，将所有结果进行合并，而如果出现部分数据源无响应的情况，会导致此处查询失败。如果相对于准确性而言，可靠性更加重要，可以为Querier添加启动参数`--partial_response`，在部分数据源响应失败时，会在日志中打印警告信息，并将该错误累加到metrics接口的计数器中，忽略错误的数据源，将现有的查询结果响应给客户端。

部分响应可在做查询时通过附加参数开启，如：partial\_response=1

部分响应分为两种策略：`warn`和`abort`。warn会忽略部分响应的错误返回状态码200和警告内容，abort会终止查询并响应错误。

### 自动下采样

在做大时间范围的查询时，可以开启自动下采样，为Querier添加启动参数`--auto-downsampling`，可选值为0，5m和1h，默认值为步长除以5。

假如步长大于25分钟时，会优先使用下采样为5m的数据做查询，加入步长大于5小时时，会优先使用下采样为1h的数据做查询。

下采样查询优先级 1h &gt; 5m &gt; 0 。

