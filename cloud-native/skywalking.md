# Skywalking

### 项目地址

{% embed url="https://github.com/apache/skywalking" %}

### 官方网站

{% embed url="https://skywalking.apache.org/" %}

### 快速上手

#### 部署skywalking

环境准备：

* k8s(配置默认storageclass)
* helm

1、安装skywalking

```
export REPO=skywalking
export SKYWALKING_RELEASE_NAME=skywalking
export SKYWALKING_RELEASE_NAMESPACE=skywalking

kubectl create ns ${SKYWALKING_RELEASE_NAMESPACE}
helm upgrade "${SKYWALKING_RELEASE_NAME}" ${REPO}/skywalking -n "${SKYWALKING_RELEASE_NAMESPACE}" --install
```

2、配置 skywalking-ui 与 skywalking-oap（如果需要从外部接入）服务 nodeport

```
apiVersion: v1
kind: Service
metadata:
  name: skywalking-ui-out
  namespace: skywalking
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
    nodePort: 31000
  selector:
    app: skywalking
    component: ui
    release: skywalking
  sessionAffinity: None
  type: NodePort
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: skywalking-oap-out
  namespace: skywalking
spec:
  ports:
  - name: grpc
    port: 11800
    protocol: TCP
    targetPort: 11800
    nodePort: 31001
  selector:
    app: skywalking
    component: oap
    release: skywalking
  sessionAffinity: None
  type: NodePort
```

3、可通过`http://<ip>:31000`端口访问skywalking控制台，外部服务链路数据上报可通过`http://<ip>:31001`

#### 部署skywalking-swck

1、下载 skywalking operator 源码

```bash
wget https://github.com/apache/skywalking-swck/archive/refs/tags/v0.6.1.tar.gz
tar zxvf v0.6.1.tar.gz && cd v0.6.1
```

2、编译构建 skywalking operator 并推送到代码仓库

<pre class="language-bash"><code class="lang-bash"># 安装 operator 代码生成工具
go install sigs.k8s.io/controller-tools/cmd/controller-gen@v0.7.0
# 编译镜像
<strong>make -C operator docker-build
</strong><strong>
</strong><strong># 推送镜像至私有仓库
</strong>export OPERATOR_IMG=&#x3C;私有仓库地址>/skywalking/controller:v0.6.1
docker tag controller:latest ${OPERATOR_IMG}
docker push ${OPERATOR_IMG}</code></pre>

3、部署 skywalking operator 并创建crds

```bash
# 部署 operator
make -C operator deploy

# 部署 crds 资源
make -C operator install
```

#### Java端接入

1、为目标容器组所在的命名空间添加标签，表示该命名空间可以使用 skywalking sidecar 自动注入

```
kubectl label ns <目标命名空间> swck-injection=enabled
```

2、为待注入的容器组资源添加注解配置

```
...
metadata:
  annotations:
    # 设置 java agent 镜像名
    sidecar.skywalking.apache.org/initcontainer.Image: "docker.io/apache/skywalking-java-agent:8.8.0-java8"
  labels:
    # 设置启用 java agent 注入
    swck-java-agent-injected: "true"
spec:
  ...
  containers:
  - ...
    env:
    - name: SW_AGENT_COLLECTOR_BACKEND_SERVICES
      value: skywalking-oap.skywalking:11800
    - name: SW_AGENT_NAME
      value: <当前容器组服务注册名称>
  ...
...
```

3、访问 skywalking-ui 控制台，从普通服务中可查询到已注册的服务

#### Golang 端接入

当前 golang 仅支持通过代码集成方式接入 skywalking，其中 go-gin 框架已实现了中间件层的适配，代码接入过程如下：

{% embed url="https://golang-2.gitbook.io/handbook/avanced/lian-lu-zhui-zong/skywalking" %}
