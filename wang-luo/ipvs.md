# IPVS

ipvs全称为IP虚拟服务器（IP Virtual Server，简写为_IPVS_）。是运行在LVS下的提供负载平衡功能的一种技术。ipvs是在linux内核netfilter模块之上实现的四层负载均衡模块。

### 开启ipvs模块

添加sysctl配置

{% code title="/etc/sysctl.conf" %}
```text
net.ipv4.ip_forward=1
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
```
{% endcode %}

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

### 安装ipvsadm

```text
yum install -y ipvsadm
```

### 运行参数

```text
ipvsadm v1.27 2008/5/15 (compiled with popt and IPVS v1.2.1)
Usage:
  ipvsadm -A|E -t|u|f service-address [-s scheduler] [-p [timeout]] [-M netmask] [--pe persistence_engine] [-b sched-flags]
  ipvsadm -D -t|u|f service-address
  ipvsadm -C
  ipvsadm -R
  ipvsadm -S [-n]
  ipvsadm -a|e -t|u|f service-address -r server-address [options]
  ipvsadm -d -t|u|f service-address -r server-address
  ipvsadm -L|l [options]
  ipvsadm -Z [-t|u|f service-address]
  ipvsadm --set tcp tcpfin udp
  ipvsadm --start-daemon state [--mcast-interface interface] [--syncid sid]
  ipvsadm --stop-daemon state
  ipvsadm -h

Commands:
Either long or short options are allowed.
  --add-service     -A        add virtual service with options
  --edit-service    -E        edit virtual service with options
  --delete-service  -D        delete virtual service
  --clear           -C        clear the whole table
  --restore         -R        restore rules from stdin
  --save            -S        save rules to stdout
  --add-server      -a        add real server with options
  --edit-server     -e        edit real server with options
  --delete-server   -d        delete real server
  --list            -L|-l     list the table
  --zero            -Z        zero counters in a service or all services
  --set tcp tcpfin udp        set connection timeout values
  --start-daemon              start connection sync daemon
  --stop-daemon               stop connection sync daemon
  --help            -h        display this help message

Options:
  --tcp-service  -t service-address   service-address is host[:port]
  --udp-service  -u service-address   service-address is host[:port]
  --fwmark-service  -f fwmark         fwmark is an integer greater than zero
  --ipv6         -6                   fwmark entry uses IPv6
  --scheduler    -s scheduler         one of rr|wrr|lc|wlc|lblc|lblcr|dh|sh|sed|nq,
                                      the default scheduler is wlc.
  --pe            engine              alternate persistence engine may be sip,
                                      not set by default.
  --persistent   -p [timeout]         persistent service
  --netmask      -M netmask           persistent granularity mask
  --real-server  -r server-address    server-address is host (and port)
  --gatewaying   -g                   gatewaying (direct routing) (default)
  --ipip         -i                   ipip encapsulation (tunneling)
  --masquerading -m                   masquerading (NAT)
  --weight       -w weight            capacity of real server
  --u-threshold  -x uthreshold        upper threshold of connections
  --l-threshold  -y lthreshold        lower threshold of connections
  --mcast-interface interface         multicast interface for connection sync
  --syncid sid                        syncid for connection sync (default=255)
  --connection   -c                   output of current IPVS connections
  --timeout                           output of timeout (tcp tcpfin udp)
  --daemon                            output of daemon information
  --stats                             output of statistics information
  --rate                              output of rate information
  --exact                             expand numbers (display exact values)
  --thresholds                        output of thresholds information
  --persistent-conn                   output of persistent connection info
  --nosort                            disable sorting output of service/server entries
  --sort                              does nothing, for backwards compatibility
  --ops          -o                   one-packet scheduling
  --numeric      -n                   numeric output of addresses and ports
  --sched-flags  -b flags             scheduler flags (comma-separated)
```

### 使用

例子：查看已有规则

```text
ipvsadm -ln
```

例子：添加一个虚拟服务192.168.1.100:80，负载到172.17.10.1:8080和172.10.10.2:8080上，以round robin为负载策略和snat转发方式。

```text
# 添加虚拟服务
ipvsadm -A -t 192.168.1.100:80 -s rr

# 添加虚拟服务上游
ipvsadm -a -t 192.168.1.100:80 -r 172.17.10.1:8080 -m
ipvsadm -a -t 192.168.1.100:80 -r 172.17.10.2:8080 -m
```

例子：将虚拟服务192.168.1.100:80的负载策略修改为wrr

```text
ipvsadm -E -t 192.168.1.100:80 -s wrr
```

例子：删除虚拟服务192.168.1.100:80

```text
ipvsadm -D -t 192.168.1.100:80
```

例子：清理所有规则

```text
ipvsadm -C
```

