# OVS

OVS通过建立gre网络隧道打通各个主机间的容器网络，其本质是在原有的报文上再封装一层gre报头，发送到目标主机上后进行解包获得实际的报文。网络传输方式如下图所示：

![https://mermaid-js.github.io/mermaid-live-editor/\#/edit/eyJjb2RlIjoiZ3JhcGggVEJcbnN1YmdyYXBoIGhvc3QwXG5wb2QwIC0tPiB2ZXRoMCh2ZXRoMCAxNzIuMTcuMTAuMi8yNClcbnBvZDEgLS0-IHZldGgxKHZldGgxIDE3Mi4xNy4xMC4zLzI0KVxudmV0aDAgLS0-IGhvc3QwX2RvY2tlcjAoZG9ja2VyMCAxNzIuMTcuMTAuMS8yNClcbnZldGgxIC0tPiBob3N0MF9kb2NrZXIwXG5ob3N0MF9kb2NrZXIwIC0tPiBob3N0MF9icjAoYnIwKVxuaG9zdDBfYnIwIC0tPiB8c3JjIDE3Mi4xNy4xMC4wLzI0IGRlc3QgMTcyLjE3LjIwLjAvMjR8IGhvc3QwX2dyZTAoZ3JlMCByZW1vdGUgaXAgMTkyLjE2OC4xLjIvMjQpXG5ob3N0MF9ncmUwIC0tPiB8Z3JlIHBhY2sgc3JjIDE5Mi4xNjguMS4xLzI0IGRlc3QgMTkyLjE2OC4xLjIvMjR8IGhvc3QwX2V0aDAoZXRoMCAxOTIuMTY4LjEuMS8yNClcbmVuZFxuXG5zdWJncmFwaCBob3N0MVxucG9kMiAtLT4gdmV0aDIodmV0aDIgMTcyLjE3LjIwLjIvMjQpXG5wb2QzIC0tPiB2ZXRoMyh2ZXRoMyAxNzIuMTcuMjAuMy8yNClcbnZldGgyIC0tPiBob3N0MV9kb2NrZXIwKGRvY2tlcjAgMTcyLjE3LjIwLjEvMjQpXG52ZXRoMyAtLT4gaG9zdDFfZG9ja2VyMFxuaG9zdDFfZG9ja2VyMCAtLT4gaG9zdDFfYnIwKGJyMClcbmhvc3QxX2JyMCAtLT4gfHNyYyAxNzIuMTcuMjAuMC8yNCBkZXN0IDE3Mi4xNy4xMC4wLzI0fCBob3N0MV9ncmUwKGdyZTAgcmVtb3RlIGlwIDE5Mi4xNjguMS4xLzI0KVxuaG9zdDFfZ3JlMCAtLT4gfGdyZSBwYWNrIHNyYyAxOTIuMTY4LjEuMiBkZXN0IDE5Mi4xNjguMS4xfCBob3N0MV9ldGgwKGV0aDAgMTkyLjE2OC4xLjIvMjQpXG5lbmRcblxuaG9zdDBfZXRoMCAtLT4gc3dpdGNoKHN3aXRjaCAxOTIuMTY4LjEuMC8yNClcbmhvc3QxX2V0aDAgLS0-IHN3aXRjaCIsIm1lcm1haWQiOnsidGhlbWUiOiJkZWZhdWx0In0sInVwZGF0ZUVkaXRvciI6ZmFsc2V9](../../../.gitbook/assets/image%20%2820%29.png)

{% hint style="info" %}
由于gre是建立点对点的网络隧道，对于每个需要联通的节点都需要创建相应的gre端口，假设节点数为n，打通所有节点间的网络需要创建n\*\(n-1\)个端口
{% endhint %}

### 跨节点通讯

1. pod的请求包经过docker0网桥到达br0
2. br0根据目标地址将请求包转入对应的gre端口
3. 转入gre端口中的请求包再封装一层gre报头，以当前物理主机的接口地址为源地址，以目标主机的接口地址为目标地址
4. 请求到达目标主机后进行解包，获得实际的请求报文
5. 目标主机将请求报文转入docker0网桥，到达目标容器

### 网络初始化

安装openvswitch

```text
yum install openvswitch -y
```

关闭selinux

{% code title="/etc/selinux/config" %}
```text
...
SELINUX=disabled
...
```
{% endcode %}

启动openvswitch服务

```text
systemctl start ovsdb-server
systemctl start ovs-vswitchd
```

创建ovs网桥

```text
ovs-vsctl add-br br0
```

创建GRE隧道端口，绑定到ovs网桥上，并指定对端主机地址

```text
ovs-vsctl add-port br0 gre0 -- set interface gre0 type=gre option:remote_ip=192.168.1.2
```

将ovs网桥接入到docker0网桥

```text
brctl addif docker0 br0
```

启动ovs与docker0网桥

```text
ip link set dev br0 up
ip link set dev docker0 up
```

将pod网段内的请求转入docker0网桥

```text
ip route add 172.17.0.0/16 dev docker0
```

源主机只知道自身的docker0地址段，并不知道其他主机上的docker0地址段，因此需要在源主机上添加一条路由规则，将所有目标为pod地址段的请求转入到docker0网桥中

