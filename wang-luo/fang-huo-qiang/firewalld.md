# firewalld

firewalld用于系统防火墙的控制，通过对iptables的底层封装简化了防火墙规则的管理。

### 查看规则

列出所有区域的规则

```text
firewall-cmd --list-all-zones
```

列出单个区域的规则

```text
firewall-cmd --list-all --zone {{ 区域名称 }}
```

所有持久化的规则都会保存在/etc/firewalld/zones/目录中，以区域名称划分文件名，格式如下：

```text
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <port protocol="tcp" port="9090"/>
  <port protocol="tcp" port="7890"/>
  <port protocol="tcp" port="7891"/>
</zone>
```

### 添加规则

例子：开启8080端口的tcp协议白名单

```text
firewall-cmd --add-port=8080/tcp --zone=public
```

这种方式开启的防火墙策略只是临时生效，当firewalld服务发生重新加载时会失效。为了使规则持久化，可以附加参数`--permanent`，当命令执行后，会在/etc/firewalld/zones/public.xml中生成对应的规则项（也可以直接修改配置文件）。这个时候规则还未在iptables中生效，通过以下命令使firewalld重新加载配置规则：

```text
firewall-cmd --reload
```

除了通过端口添加规则外，还可通过服务名添加规则

例子：开启sshd服务的白名单访问

```text
firewall-cmd --add-service=ssh --zone=public --permanent
```

### 移除规则

例子：移除本地8080端口的tcp规则

```text
firewall-cmd --delete-port=8080/tcp --zone=public --permanent
firewall-cmd --reload
```

