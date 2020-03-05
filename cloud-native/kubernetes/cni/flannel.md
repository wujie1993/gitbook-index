# Flannel

flannel通过建立overlay网络实现了容器的跨节点通讯，请求包由flanneld进行vxlan封包发送和解包。网络传输方式如下图所示：

![](../../../.gitbook/assets/image%20%2815%29.png)

原图链接

## 工作流程

### 网络初始化

1. 在etcd中为flannel分配pod地址段（如：172.17.0.0/16）；
2. 为flanneld配置etcd的访问地址，启动flanneld服务；
3. flanneld获取到当前节点对应的pod子地址段（如：172.17.10.0/24），将该子地址段设置到docker服务的启动参数中并启动docker服务；
4. docker以设定的容器网段初始化docker0网桥；
5. flanneld创建flannel0网桥，一端接到docker0网桥上，另一端接到flanneld进程上；
6. 在pod创建时docker为其分配一个节点子网段的地址，并且以flannel0网桥的地址作为默认路由；
7. flanneld从etcd同步每个pod的地址信息在内存中生成路由表。

### 容器跨节点通讯

1. pod依据默认路由将请求包从veth通过docker0网桥发送到对端的flannel0网桥上；
2. 发送到flannel0网桥上的请求会由用户态的flanneld进程抓取并进行vxlan封包；
3. flanneld根据内存中的路由表获取到目的pod所在的节点地址，将请求包发送到目的节点上；
4. 目的节点的flanneld从物理接口上获取到请求包后进行解包，获得pod的目标地址并发送到对应的pod上。

