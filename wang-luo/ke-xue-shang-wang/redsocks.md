# Redsocks

Redsocks 可以通过 iptables 将 TCP 连接重定向至 SOCKS 或 HTTPS 代理，实现 TCP/IP 传输层代理。

### 快速上手

1、安装源码编译所需的依赖包

```
yum install libevent-devel git gcc -y
```

2、拉取源代码

```
git clone http://github.com/darkk/redsocks.git
```

3、编译安装 redsocks 可执行文件

```
cd redsocks && make && cp redsocks /usr/bin/
```

4、创建配置文件

{% code title="/etc/redsocks.conf" %}
```
base {
        log_debug = off;
        log_info = on;
        log = stderr;
        daemon = off;
        redirector = iptables;
}
redsocks {
        # 本地服务地址
        local_ip = 127.0.0.1;
        # 本地服务端口
        local_port = 8888;
        # 远端代理服务地址
        ip = 127.0.0.1;
        # 远端代理服务端口
        port = 1080;
        # 远端代理服务协议
        type = socks5;
}
```
{% endcode %}

5、启动服务

```
redsocks -c /etc/redsocks.conf
```

6、配置本机 iptables 代理规则

```
#!/bin/bash
# 创建 REDSOCKS nat 转发链
iptables -t nat -N REDSOCKS

# 忽略部分保留地址，不进行转发，参考 http://www.iana.org/assignments/iana-ipv4-special-registry/iana-ipv4-special-registry.xhtml
iptables -t nat -A REDSOCKS -d 0.0.0.0/8 -j RETURN
iptables -t nat -A REDSOCKS -d 10.0.0.0/8 -j RETURN
iptables -t nat -A REDSOCKS -d 127.0.0.0/8 -j RETURN
iptables -t nat -A REDSOCKS -d 169.254.0.0/16 -j RETURN
iptables -t nat -A REDSOCKS -d 172.16.0.0/12 -j RETURN
iptables -t nat -A REDSOCKS -d 192.168.0.0/16 -j RETURN
iptables -t nat -A REDSOCKS -d 224.0.0.0/4 -j RETURN
iptables -t nat -A REDSOCKS -d 240.0.0.0/4 -j RETURN

# 将 REDSOCKS 链所接收到的 TCP 请求重定向至 8888 端口
iptables -t nat -A REDSOCKS -p tcp -j REDIRECT --to-ports 8888

# 设置自定义转发策略，将目的为 443 和 80 端口的请求走 REDSOCKS 链代理转发
iptables -t nat -A OUTPUT -p tcp --dport 443 -j REDSOCKS
iptables -t nat -A OUTPUT -p tcp --dport 80 -j REDSOCKS
```
