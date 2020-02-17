# Config

Config组件提供了一套多租户的接口方法用来处理各种用于prometheus的配置文件。租户可以在数据库中读取和写入Prometheus规则文件，Alertmanager配置文件和Alertmanager模板。

每个租户将拥有自己的一组规则文件，Alertmanager配置和模板。

配置存储的数据结构：

```text
{
    "id": 99,
    "rule_format_version": "2",
    "config": {
        "alertmanager_config": "<standard alertmanager.yaml config>",
        "rules_files": {
            "rules.yaml": "<standard rules.yaml config>",
            "rules2.yaml": "<standard rules.yaml config>"
         },
        "template_files": {
            "templates.tmpl": "<standard template file>",
            "templates2.tmpl": "<standard template file>"
        }
    }
}
```

* `id`。每次配置发生更新时都会增长，Cortex会使用编号最大的配置
*  `rule_format_version` 。允许兼容的prometheus版本，可选值有1和2。
* `config.alertmanager_config`。alertmanager的配置，将内容编码为单个json字符串。
* `config.rules_files`。存放recording和alerting规则配置，将内容编码为单个json字符串。
* `config.template_files`。存放alertmanager的告警通知模板，将内容编码为单个json字符串。

### 管理接口

* GET /api/prom/configs/alertmanager。获取Alertmanager配置
* POST /api/prom/configs/alertmanager。更新Alertmanager配置
* POST /api/prom/configs/alertmanager/validate。验证Alertmanager配置
* GET /api/prom/configs/rules。获取规则配置
* POST /api/prom/configs/rules。更新规则配置
* GET /api/prom/configs/templates。获取模板配置
* POST /api/prom/configs/templates。更新模板配置
* DELETE /api/prom/configs/deactivate。禁用一个租户的配置
* POST /api/prom/configs/restore。恢复一个租户的配置

