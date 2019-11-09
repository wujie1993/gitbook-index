# IPVS

## 开启ipvs

添加sysctl配置

{% tabs %}
{% tab title="/etc/sysctl.conf" %}
```text
net.ipv4.ip_forward=1
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
```
{% endtab %}
{% endtabs %}

使配置生效

```text
sysctl -p
```

加载内核模块

```text
modprobe ip_vs
modprobe ip_vs_rr
modprobe ip_vs_wrr
modprobe ip_vs_sh
modprobe nf_conntrack_ipv4
```

