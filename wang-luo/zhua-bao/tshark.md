# tshark

由wireshark开源的网络抓包工具

### 安装

```text
yum install wireshark -y
```

### 使用

例子：抓取eth0接口上的gre报文

```text
tshark -i eth0 -R ip proto GRE
```



