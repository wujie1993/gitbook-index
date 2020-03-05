# Calico

Calico采用的是纯三层的网络方案，通过BGP协议将到达各个节点docker0网桥的路由进行组播分发，在容器进行跨节点通讯时，直接通过路由的方式发送请求包，不做任何的封装处理。基于此，Calico的网络报文传输相较于Overlay网络更加高效。网络传输方式如下图所示：

![](../../../.gitbook/assets/image%20%2826%29.png)

### 项目地址

{% embed url="https://github.com/projectcalico/calico" %}

### 官方文档

{% embed url="https://docs.projectcalico.org/introduction/" %}

### 结构图

![](../../../.gitbook/assets/image%20%281%29.png)

### 组件说明

* **Felix**。以守护进程的方式运行于每个容器节点上，为该节点初始化IP信息，路由规则和访问控制策略（通过iptables），并且将节点的健康状态上报至etcd。
* **BGP client \(BIRD\)**。做为BGP客户端与Felix一同部署于容器节点上。当Felix将路由信息写入到内核路由表中时，BIRD会获取并通过BGP路由分发协议将路由信息分发到其他节点的BIRD上。实现跨节点的容器网络互通。
* **BGP route reflector \(BIRD\)。**做为BGP路由分发的中心节点。由于每个BGP客户端会将路由信息分发给集群中的其他所有BGP客户端，这意味着当集群中BGP客户端数量为n时，需要建立n\*n个网络拓扑连接。可以将BGP route reflector部署m个，这样其他的BGP客户端都会通过这m个BGP route reflector分发路由信息，总共只需要m\*n个网络拓扑连接，降低路由协议分发所带来的网络压力。

### 快速开始

{% embed url="https://docs.projectcalico.org/getting-started/kubernetes/installation/calico" %}

### 相关资料

{% embed url="https://my.oschina.net/u/3390908/blog/1649764" %}





