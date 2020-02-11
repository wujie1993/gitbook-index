# Store

Store组件实现了用于查询对象存储桶中历史数据的Store API，在服务启动后会将少量的历史数据信息（如：block index）缓存到本地并定期更新，用以提升查询速度，这部分数据是可删除的。

### 可用参数

```text
usage: thanos store [<flags>]

store node giving access to blocks in a bucket provider. Now supported GCS, S3,
Azure, Swift and Tencent COS.

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
      --data-dir="./data"        Data directory in which to cache remote blocks.
      --index-cache-size=250MB   Maximum size of items held in the in-memory
                                 index cache. Ignored if --index-cache.config or
                                 --index-cache.config-file option is specified.
      --index-cache.config-file=<file-path>
                                 Path to YAML file that contains index cache
                                 configuration. See format details:
                                 https://thanos.io/components/store.md/#index-cache
      --index-cache.config=<content>
                                 Alternative to 'index-cache.config-file' flag
                                 (lower priority). Content of YAML file that
                                 contains index cache configuration. See format
                                 details:
                                 https://thanos.io/components/store.md/#index-cache
      --chunk-pool-size=2GB      Maximum size of concurrently allocatable bytes
                                 reserved strictly to reuse for chunks in
                                 memory.
      --store.grpc.series-sample-limit=0
                                 Maximum amount of samples returned via a single
                                 Series call. 0 means no limit. NOTE: For
                                 efficiency we take 120 as the number of samples
                                 in chunk (it cannot be bigger than that), so
                                 the actual number of samples might be lower,
                                 even though the maximum could be hit.
      --store.grpc.series-max-concurrency=20
                                 Maximum number of concurrent Series calls.
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
      --sync-block-duration=3m   Repeat interval for syncing the blocks between
                                 local and remote view.
      --block-sync-concurrency=20
                                 Number of goroutines to use when constructing
                                 index-cache.json blocks from object storage.
      --min-time=0000-01-01T00:00:00Z
                                 Start of time range limit to serve. Thanos
                                 Store will serve only metrics, which happened
                                 later than this value. Option can be a constant
                                 time in RFC3339 format or time duration
                                 relative to current time, such as -1d or 2h45m.
                                 Valid duration units are ms, s, m, h, d, w, y.
      --max-time=9999-12-31T23:59:59Z
                                 End of time range limit to serve. Thanos Store
                                 will serve only blocks, which happened eariler
                                 than this value. Option can be a constant time
                                 in RFC3339 format or time duration relative to
                                 current time, such as -1d or 2h45m. Valid
                                 duration units are ms, s, m, h, d, w, y.
      --selector.relabel-config-file=<file-path>
                                 Path to YAML file that contains relabeling
                                 configuration that allows selecting blocks. It
                                 follows native Prometheus relabel-config
                                 syntax. See format details:
                                 https://prometheus.io/docs/prometheus/latest/configuration/configuration/#relabel_config
      --selector.relabel-config=<content>
                                 Alternative to 'selector.relabel-config-file'
                                 flag (lower priority). Content of YAML file
                                 that contains relabeling configuration that
                                 allows selecting blocks. It follows native
                                 Prometheus relabel-config syntax. See format
                                 details:
                                 https://prometheus.io/docs/prometheus/latest/configuration/configuration/#relabel_config
```

### 时间分片

thanos store在做查询时会检索指定时间区间内的所有数据块，当时间区间较大时，查询可能会花费大量时间。为了解决这一问题，在启动时可以附加参数`--min-time`和`--max-time`（如`--min-time=-1w`和`--max-time=-2w`表示只查询一周前到两周前这段区间的历史数据），用以界定其所负责查询的时间区间，超过该时间区间的部分会被忽略，通过将时间区间拆分给多个store负责查询可以降低单个store服务做大范围时间区间查询时的压力并提升查询速度。

store在更新时间区间时会有3分钟左右的误差，为了避免在查询时遗漏时间界限前后的数据，建议在设置多个store服务的时间区间时有一定程度的重叠，query从不同的store服务获取到历史数据后会进行去重处理。

store在查询历史数据时是以trunk为单位，因此获取到的数据有可能会超过`--min-time`和`--max-time`的设定范围。

### 索引缓存

store组件支持索引缓存以加速时间序列的检索查询速度。目前支持两种方式：内存和Memcached。

内存索引缓存默认会开启，通过参数`--index-cache-size`可以指定缓存数据量的大小。也可以通过`--index-cache.config-file`参数指定配置文件路径并在配置文件中指定，如：

```text
type: IN-MEMORY
config:
  max_size: 0
  max_item_size: 0
```

Memcached索引缓存需要指定memcached服务做为缓存后端，通过`--index-cache.config-file`参数指定配置文件路径并在配置文件中指定，如：

```text
type: MEMCACHED
config:
  addresses: []
  timeout: 0s
  max_idle_connections: 0
  max_async_concurrency: 0
  max_async_buffer_size: 0
  max_get_multi_concurrency: 0
  max_get_multi_batch_size: 0
  dns_provider_update_interval: 0s
```

其中.config.addresses为必填项，指定memcached服务地址

