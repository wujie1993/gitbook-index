# firewalld

firewalld用于系统防火墙的控制，通过对iptables的底层封装简化了防火墙规则的管理。

列出所有区域的规则

```text
firewall-cmd --list-all-zones
```

列出单个区域的规则

```text
firewall-cmd --list-all --zone {{ 区域名称 }}
```

例子：开启8080端口的tcp协议白名单

```text
firewall-cmd --add-port=8080/tcp --zone=public
```

这种方式开启的防火墙策略只是临时生效，当firewalld服务发生重新加载时会失效。为了使规则持久化，可以附加参数`--permanent`，当命令执行后，会在/etc/firewalld/zones/public.xml中生成对应的规则项。这个时候规则还未在iptables中生效，通过以下命令使firewalld重新加载配置规则：

```text
firewall-cmd --reload
```



