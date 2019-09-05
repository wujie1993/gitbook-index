# Dnsmasq

## 介绍

_Dnsmasq_ 提供DNS 缓存和DHCP 服务功能。作为域名解析服务器\(DNS\)，_dnsmasq_可以通过缓存DNS 请求来提高对访问过的网址的连接速度。

## 安装与配置

安装_dnsmasq_

```bash
yum install dnsmasq -y
```

配置文件在/etc/dnsmasq.conf，/etc/dnsmasq.d/目录中存放的是一些自定义配置

配置侦听地址与端口

```text
# dns服务侦听端口，默认就是53，可不配
port=53
# dns服务侦听接口，如果需要侦听多个接口，可重复配置多条
interface={{ 本机网络接口 }}
# dns服务侦听地址，如果需要侦听多个地址，可重复配置多条
listen-address={{ 本机ip地址 }}
```

启动_dnsmasq_服务并配置开机自启动

```bash
systemctl start dnsmasq
systemctl enable dnsmasq
```

服务开启成功后默认会侦听到53/udp端口上

修改/etc/resolv.conf，将系统的域名解析服务地址指向当前机器上的dnsmasq服务

{% code-tabs %}
{% code-tabs-item title="/etc/resolv.conf" %}
```text
nameserver {{ 本机ip地址 }}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

这时如果访问外部域名是不通的，因为还没有添加解析策略，需要配置upstream使域名解析指向上游的dns服务

### 上游DNS

添加上游dns配置文件/etc/dnsmasq.d/public\_upstream.conf，添加以下配置项

{% code-tabs %}
{% code-tabs-item title="/etc/dnsmasq.d/public\_upstream.conf" %}
```text
# 上游dns服务器地址配置文件路径
resolv-file=/etc/resolv.dnsmasq.conf
# 使上游dns服务器查询按照配置文件中的顺序查询而不是同时查询
strict-order
```
{% endcode-tabs-item %}
{% endcode-tabs %}

添加上游dns服务地址配置文件

{% code-tabs %}
{% code-tabs-item title="/etc/resolv.dnsmasq.conf" %}
```text
nameserver 8.8.8.8
nameserver 114.114.114.114
```
{% endcode-tabs-item %}
{% endcode-tabs %}

重启_dnsmasq_服务使配置生效

```bash
systemctl restart dnsmasq
```

### 自定义域名

添加自定义域名解析配置文件/etc/dnsmasq.d/local\_host.conf，添加以下配置项

{% code-tabs %}
{% code-tabs-item title="/etc/dnsmasq.d/local\_host.conf" %}
```text
# 不使用/etc/hosts中的解析配置
no-hosts
# 将指定域名解析为指定的地址，可添加多条策略
address=/{{ 域名 }}/{{ ip地址 }}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

例子：泛域名解析

```text
# 解析*.example.com到192.168.1.1
address=/.example.com/192.168.1.1
```

重启_dnsmasq_服务使配置生效

```bash
systemctl restart dnsmasq
```



