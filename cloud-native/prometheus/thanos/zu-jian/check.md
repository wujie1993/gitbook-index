# Check

Check组件用于Prometheus Rules的规则校验。在Thanos rule组件中使用了该组件进行规则校验。可以看作是`promtool check rules`的拓展，新增了对于 `partial_response_strategy` 字段的校验。

如果检查失败命令行会返回退出码Exit Code\(1\)，成功返回Exit Code\(0\)

### 可选参数

```text
usage: thanos check rules <rule-files>...

Check if the rule files are valid or not.

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

Args:
  <rule-files>  The rule files to check.
```



