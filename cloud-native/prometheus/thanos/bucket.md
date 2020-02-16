# Bucket

Thanos的bucket组件是一个用于查看对象存储桶中数据内容的命令集合。它通常作為獨立命令運行，以幫助進行故障排除。

例子：

```text
$ thanos bucket verify --objstore.config-file=bucket.yml
```

bucket.yml内容：

```text
type: GCS
config:
  bucket: example-bucket
```

可以在`/cmd/thanos/bucket.go`添加新的命令以用于帮助我们更好地使用对象存储桶。

## 使用

### 可用参数

```text
usage: thanos bucket [<flags>] <command> [<args> ...]

Bucket utility commands

Flags:
  -h, --help               Show context-sensitive help (also try --help-long and
                           --help-man).
      --version            Show application version.
      --log.level=info     Log filtering level.
      --log.format=logfmt  Log format to use. Possible options: logfmt or json.
      --tracing.config-file=<file-path>
                           Path to YAML file with tracing configuration. See
                           format details:
                           https://thanos.io/tracing.md/#configuration
      --tracing.config=<content>
                           Alternative to 'tracing.config-file' flag (lower
                           priority). Content of YAML file with tracing
                           configuration. See format details:
                           https://thanos.io/tracing.md/#configuration
      --objstore.config-file=<file-path>
                           Path to YAML file that contains object store
                           configuration. See format details:
                           https://thanos.io/storage.md/#configuration
      --objstore.config=<content>
                           Alternative to 'objstore.config-file' flag (lower
                           priority). Content of YAML file that contains object
                           store configuration. See format details:
                           https://thanos.io/storage.md/#configuration

Subcommands:
  bucket verify [<flags>]
    Verify all blocks in the bucket against specified issues

  bucket ls [<flags>]
    List all blocks in the bucket

  bucket inspect [<flags>]
    Inspect all blocks in the bucket in detailed, table-like way

  bucket web [<flags>]
    Web interface for remote storage bucket
```

### web

`bucket web`模块提供了可交互的web界面用于查看对象存储桶中的数据块。服务启动后会定期地刷新并更新视图内容。

![](https://thanos.io/img/bucket-web.jpg)

显示列表分为多行，每一行表示一个数据推送源实例（如sidecar或rule）。每行又分为多个块，每个块的不同颜色表示不同的压缩级别和解析度，块的长度表示其存储数据的时间区间长度。

例子：`$ thanos bucket web --objstore.config-file="..."`

```text
usage: thanos bucket web [<flags>]

Web interface for remote storage bucket

Flags:
  -h, --help                    Show context-sensitive help (also try
                                --help-long and --help-man).
      --version                 Show application version.
      --log.level=info          Log filtering level.
      --log.format=logfmt       Log format to use. Possible options: logfmt or
                                json.
      --tracing.config-file=<file-path>
                                Path to YAML file with tracing configuration.
                                See format details:
                                https://thanos.io/tracing.md/#configuration
      --tracing.config=<content>
                                Alternative to 'tracing.config-file' flag (lower
                                priority). Content of YAML file with tracing
                                configuration. See format details:
                                https://thanos.io/tracing.md/#configuration
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
      --http-address="0.0.0.0:10902"
                                Listen host:port for HTTP endpoints.
      --http-grace-period=2m    Time to wait after an interrupt received for
                                HTTP Server.
      --web.external-prefix=""  Static prefix for all HTML links and redirect
                                URLs in the bucket web UI interface. Actual
                                endpoints are still served on / or the
                                web.route-prefix. This allows thanos bucket web
                                UI to be served behind a reverse proxy that
                                strips a URL sub-path.
      --web.prefix-header=""    Name of HTTP request header used for dynamic
                                prefixing of UI links and redirects. This option
                                is ignored if web.external-prefix argument is
                                set. Security risk: enable this option only if a
                                reverse proxy in front of thanos is resetting
                                the header. The
                                --web.prefix-header=X-Forwarded-Prefix option
                                can be useful, for example, if Thanos UI is
                                served via Traefik reverse proxy with
                                PathPrefixStrip option enabled, which sends the
                                stripped prefix value in X-Forwarded-Prefix
                                header. This allows thanos UI to be served on a
                                sub-path.
      --refresh=30m             Refresh interval to download metadata from
                                remote storage
      --timeout=5m              Timeout to download metadata from remote storage
      --label=LABEL             Prometheus label to use as timeline title
```

### verify

`bucket verify`用于验证和选择性地修复指定存储桶中的数据块

例子：`thanos bucket verify --objstore.config-file="..."`

```text
usage: thanos bucket verify [<flags>]

Verify all blocks in the bucket against specified issues

Flags:
  -h, --help               Show context-sensitive help (also try --help-long and
                           --help-man).
      --version            Show application version.
      --log.level=info     Log filtering level.
      --log.format=logfmt  Log format to use. Possible options: logfmt or json.
      --tracing.config-file=<file-path>
                           Path to YAML file with tracing configuration. See
                           format details:
                           https://thanos.io/tracing.md/#configuration
      --tracing.config=<content>
                           Alternative to 'tracing.config-file' flag (lower
                           priority). Content of YAML file with tracing
                           configuration. See format details:
                           https://thanos.io/tracing.md/#configuration
      --objstore.config-file=<file-path>
                           Path to YAML file that contains object store
                           configuration. See format details:
                           https://thanos.io/storage.md/#configuration
      --objstore.config=<content>
                           Alternative to 'objstore.config-file' flag (lower
                           priority). Content of YAML file that contains object
                           store configuration. See format details:
                           https://thanos.io/storage.md/#configuration
      --objstore-backup.config-file=<file-path>
                           Path to YAML file that contains object store-backup
                           configuration. See format details:
                           https://thanos.io/storage.md/#configuration Used for
                           repair logic to backup blocks before removal.
      --objstore-backup.config=<content>
                           Alternative to 'objstore-backup.config-file' flag
                           (lower priority). Content of YAML file that contains
                           object store-backup configuration. See format
                           details: https://thanos.io/storage.md/#configuration
                           Used for repair logic to backup blocks before
                           removal.
  -r, --repair             Attempt to repair blocks for which issues were
                           detected
  -i, --issues=index_issue... ...
                           Issues to verify (and optionally repair). Possible
                           values: [duplicated_compaction index_issue
                           overlapped_blocks]
      --id-whitelist=ID-WHITELIST ...
                           Block IDs to verify (and optionally repair) only. If
                           none is specified, all blocks will be verified.
                           Repeated field
```

### ls

`bucket ls`用于列出指定存储中的所有数据块

例子：`$ thanos bucket ls -o json --objstore.config-file="..."`

```text
usage: thanos bucket ls [<flags>]

List all blocks in the bucket

Flags:
  -h, --help               Show context-sensitive help (also try --help-long and
                           --help-man).
      --version            Show application version.
      --log.level=info     Log filtering level.
      --log.format=logfmt  Log format to use. Possible options: logfmt or json.
      --tracing.config-file=<file-path>
                           Path to YAML file with tracing configuration. See
                           format details:
                           https://thanos.io/tracing.md/#configuration
      --tracing.config=<content>
                           Alternative to 'tracing.config-file' flag (lower
                           priority). Content of YAML file with tracing
                           configuration. See format details:
                           https://thanos.io/tracing.md/#configuration
      --objstore.config-file=<file-path>
                           Path to YAML file that contains object store
                           configuration. See format details:
                           https://thanos.io/storage.md/#configuration
      --objstore.config=<content>
                           Alternative to 'objstore.config-file' flag (lower
                           priority). Content of YAML file that contains object
                           store configuration. See format details:
                           https://thanos.io/storage.md/#configuration
  -o, --output=""          Optional format in which to print each block's
                           information. Options are 'json', 'wide' or a custom
                           template.
```

### inspect 

bucket inspect用于将存储桶中的指定数据块信息以表格形式打印到标准输出控制台中

例子：$ thanos bucket inspect -l environment=\"prod\" --objstore.config-file="..."

```text
usage: thanos bucket inspect [<flags>]

Inspect all blocks in the bucket in detailed, table-like way

Flags:
  -h, --help                 Show context-sensitive help (also try --help-long
                             and --help-man).
      --version              Show application version.
      --log.level=info       Log filtering level.
      --log.format=logfmt    Log format to use. Possible options: logfmt or
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
      --objstore.config-file=<file-path>
                             Path to YAML file that contains object store
                             configuration. See format details:
                             https://thanos.io/storage.md/#configuration
      --objstore.config=<content>
                             Alternative to 'objstore.config-file' flag (lower
                             priority). Content of YAML file that contains
                             object store configuration. See format details:
                             https://thanos.io/storage.md/#configuration
  -l, --selector=<name>=\"<value>\" ...
                             Selects blocks based on label, e.g. '-l
                             key1=\"value1\" -l key2=\"value2\"'. All key value
                             pairs must match.
      --sort-by=FROM... ...  Sort by columns. It's also possible to sort by
                             multiple columns, e.g. '--sort-by FROM --sort-by
                             UNTIL'. I.e., if the 'FROM' value is equal the rows
                             are then further sorted by the 'UNTIL' value.
      --timeout=5m           Timeout to download metadata from remote storage
```





