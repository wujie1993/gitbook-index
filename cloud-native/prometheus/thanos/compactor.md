# Compactor

Compactor用于定时对远端对象存储中的历史数据块进行下采样，提升在做大时间跨度查询时的速度。由于Compactor的设计是非并发安全的，因此只能单例部署。

### 下采样

下采样有三个主要的配置参数：

* `--retention.resolution-raw`（单位：d，默认0d）
* `--retention.resolution-5m`（单位：d，默认0d）
* `--retention.resolution-1h`（单位：d，默认0d）

当开启raw之后，原有的历史数据会以该项所配置的时间长度保留于远端对象存储中，超过该时间的数据会被清理。

5m开启后会为每个存储时长大于**40小时**的块中开辟新的存储区域，将历史数据以5分钟为精度进行下采样，以该项所配置的时间长度存储于远端对象存储中。

1h开启后会为每个存储时长大于**10天**的块中开辟新的存储区域，将历史数据以1小时为间隔进行下采样，以该项所配置的时间长度存储于远端对象存储中。

这三种采样的数据都是独立存储，相当于存了三份数据，因此并不能起到压缩存储空间的作用。默认情况下值为0d表示永久保留下采样数据。如果需要关闭下采样，也可以在启动时附加`--debug.disable-downsampling`参数

下采样的实现方式是在原有的历史数据中以采样精度为取值区间取一个指标值保存到下采样数据块中。

### 可选参数

```text
usage: thanos compact [<flags>]

continuously compacts blocks in an object store bucket

Flags:
  -h, --help                   Show context-sensitive help (also try --help-long
                               and --help-man).
      --version                Show application version.
      --log.level=info         Log filtering level.
      --log.format=logfmt      Log format to use. Possible options: logfmt or
                               json.
      --tracing.config-file=<file-path>
                               Path to YAML file with tracing configuration. See
                               format details:
                               https://thanos.io/tracing.md/#configuration
      --tracing.config=<content>
                               Alternative to 'tracing.config-file' flag (lower
                               priority). Content of YAML file with tracing
                               configuration. See format details:
                               https://thanos.io/tracing.md/#configuration
      --http-address="0.0.0.0:10902"
                               Listen host:port for HTTP endpoints.
      --http-grace-period=2m   Time to wait after an interrupt received for HTTP
                               Server.
      --data-dir="./data"      Data directory in which to cache blocks and
                               process compactions.
      --objstore.config-file=<file-path>
                               Path to YAML file that contains object store
                               configuration. See format details:
                               https://thanos.io/storage.md/#configuration
      --objstore.config=<content>
                               Alternative to 'objstore.config-file' flag (lower
                               priority). Content of YAML file that contains
                               object store configuration. See format details:
                               https://thanos.io/storage.md/#configuration
      --consistency-delay=30m  Minimum age of fresh (non-compacted) blocks
                               before they are being processed. Malformed blocks
                               older than the maximum of consistency-delay and
                               48h0m0s will be removed.
      --retention.resolution-raw=0d
                               How long to retain raw samples in bucket. Setting
                               this to 0d will retain samples of this resolution
                               forever
      --retention.resolution-5m=0d
                               How long to retain samples of resolution 1 (5
                               minutes) in bucket. Setting this to 0d will
                               retain samples of this resolution forever
      --retention.resolution-1h=0d
                               How long to retain samples of resolution 2 (1
                               hour) in bucket. Setting this to 0d will retain
                               samples of this resolution forever
  -w, --wait                   Do not exit after all compactions have been
                               processed and wait for new work.
      --downsampling.disable   Disables downsampling. This is not recommended as
                               querying long time ranges without non-downsampled
                               data is not efficient and useful e.g it is not
                               possible to render all samples for a human eye
                               anyway
      --block-sync-concurrency=20
                               Number of goroutines to use when syncing block
                               metadata from object storage.
      --compact.concurrency=1  Number of goroutines to use when compacting
                               groups.
      --selector.relabel-config-file=<file-path>
                               Path to YAML file that contains relabeling
                               configuration that allows selecting blocks. It
                               follows native Prometheus relabel-config syntax.
                               See format details:
                               https://prometheus.io/docs/prometheus/latest/configuration/configuration/#relabel_config
      --selector.relabel-config=<content>
                               Alternative to 'selector.relabel-config-file'
                               flag (lower priority). Content of YAML file that
                               contains relabeling configuration that allows
                               selecting blocks. It follows native Prometheus
                               relabel-config syntax. See format details:
                               https://prometheus.io/docs/prometheus/latest/configuration/configuration/#relabel_config
```

### 在Store做查询时在什么条件下会使用到下采样的数据



