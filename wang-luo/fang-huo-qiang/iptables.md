# iptables

例子：添加对地址192.168.1.1的白名单

```text
iptables -A INPUT -s 192.168.1.1 -p all -j ACCEPT
```

例子：插入对地址192.168.1.1的白名单

```text
iptables -I INPUT -s 192.168.1.1 -p all -j ACCEPT
```

例子：添加对来自地址192.168.1.1的传入白名单

```text
iptables -A INPUT -s 192.168.1.1 -p INPUT -j ACCEPT
```

例子：添加对来自地址192.168.1.1的传出白名单

```text
iptables -A INPUT -s 192.168.1.1 -p OUTPUT -j ACCEPT
```

例子：添加对来自地址192.168.1.1的转发白名单

```text
iptables -A INPUT -s 192.168.1.1 -p FORWARD -j ACCEPT
```

例子：删除对地址192.168.1.1的白名单

```text
iptables -D INPUT -s 192.168.1.1 -p all -j ACCEPT
```

例子：添加对地址192.168.1.1的黑名单

```text
iptables -A INPUT -s 192.168.1.1 -p all -j DROP
```



