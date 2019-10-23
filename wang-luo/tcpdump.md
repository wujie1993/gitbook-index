# tcpdump

例子：监听eth0接口

```text
tcpdump -i eth0
```

例子：监听192.168.1.1地址，包括来源与目的

```text
tcpdump host 127.0.0.1
```

例子：监听来自于192.168.1.1的数据包

```text
tcpdump src host 192.168.1.1
```

例子：监听发送到192.168.1.1的数据包

```text
tcpdump dst host 192.168.1.1
```

例子：监听8080端口，包括来源与目的

```text
tcpdump port 8080
```

例子：监听来自于8080端口的数据包

```text
tcpdump src port 8080
```

例子：监听发送到8080的数据包

```text
tcpdump dst port 8080
```

例子：监听tcp包

```text
tcpdump tcp
```

例子：监听udp包

```text
tcpdump udp
```

例子：监听发送到8080端口，来自于192.168.1.1地址的tcp包

```text
tcpdump tcp dst port 8080 and src 192.168.1.1
```

