---
description: 部署生产环境就绪的kubernetes集群
---

# kubespray

## 项目地址

{% embed url="https://github.com/kubernetes-sigs/kubespray" %}

## 快速开始

前置步骤：

* 关闭防火墙或配置服务器间的互信任策略
* 系统时钟同步
* ansible 节点到 k8s 节点的 ssh 免密登录

1、下载kubespray

```text
git@github.com:kubernetes-sigs/kubespray.git && cd kubespray && git checkout release-2.15
```

2、安装python3与pip3

**ubuntu环境**

```text
sudo apt install python3
```

**centos7环境**

```text
sudo yum install python3
```

3、安装python依赖包

```text
python3 -m pip install -r requirements.txt
```

{% hint style="info" %}
如果已经通过其他方式安装过ansible，请先执行卸载，以确保ansible使用的是python3
{% endhint %}

4、复制示例配置

```text
cp -rfp inventory/sample inventory/mycluster
```

5、编辑inventory清单

```text
cat inventory/mycluster/inventory.ini
```

6、查看并编辑group\_vars参数

```text
cat inventory/mycluster/group_vars/all/all.yml
cat inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml
```

7、执行安装

```text
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml
```

{% hint style="info" %}
拉取k8s.gcr.io的镜像时可能会出错，建议预先从其他的镜像地址中先拉取好，在inventory/mycluster/hosts.yaml中配置download\_always\_pull: false避免重复拉取已经存在的镜像；

拉取gcr.io的镜像可以配置国内镜像源gcr\_image\_repo: registry.aliyuncs.com/
{% endhint %}

8、查看集群、节点以及 Pod 状态

```text
kubectl cluster-info
kubectl get node -o wide
kubectl get pod -o wide
```

## 组件安装

### kube-dashboard

1、配置 dashboard

{% code title="inventory/mycluster/group\_vars/k8s-cluster/addons.yml" %}
```text
dashboard_enabled: true
dashboard_token_ttl: 43200
```
{% endcode %}

2、安装 dashboard

```text
ansible-playbook -i inventory/mycluster/hosts.yaml --tags "apps" --become --become-user=root cluster.yml
```

3、通过 NodePort 方式暴露 dashboard 服务

```text
kubectl edit svc kubernetes-dashboard -n kube-system
```

```text
apiVersion: v1
kind: Service
metadata:
  ...
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  ...
  ports:
  - nodePort: 30000
    port: 443
    protocol: TCP
    targetPort: 8443
  ...
  type: NodePort

```

4、创建集群管理用户

```text
kubectl create sa admin -n kube-system
kubectl create clusterrolebinding --clusterrole='cluster-admin' --serviceaccount=kube-system:admin admin
```

5、通过浏览器访问 proxy 节点的 30000 端口，使用下方命令获取 token 登录 dashboard

```text
kubectl get secret -n kube-system | grep admin | awk '{print $1}' | xargs kubectl -n kube-system get secret -o jsonpath='{.data.token}' | base64 --decode
```

### metrics-server

1、配置 metrics-server

{% code title="inventory/mycluster/group\_vars/k8s-cluster/addons.yml" %}
```text
metrics_server_enabled: true
```
{% endcode %}

2、安装 metrics-server

```text
ansible-playbook -i inventory/mycluster/hosts.yaml --tags "apps" --become --become-user=root cluster.yml
```

3、访问 dashboard 访问 pod 可看到运行指标

### helm

1、配置 helm

{% code title="inventory/mycluster/group\_vars/k8s-cluster/addons.yml" %}
```text
helm_enabled: true
```
{% endcode %}

2、安装 helm

```text
ansible-playbook -i inventory/mycluster/hosts.yaml --tags "apps" --become --become-user=root cluster.yml
```

3、在 \[kube-master\] 节点上可执行 helm 指令

### local-path-provisioner

1、配置 local-path-provisioner

{% code title="inventory/mycluster/group\_vars/k8s-cluster/addons.yml" %}
```text
local_path_provisioner_enabled: true
local_path_provisioner_claim_root: /data/k8s/local-path-provisioner/
local_path_provisioner_image_tag: "v0.0.17"
```
{% endcode %}

2、安装 local-path-provisioner

```text
ansible-playbook -i inventory/mycluster/hosts.yaml --tags "apps" --become --become-user
```

## 集群配置

### 设置 dns上游

1、配置 dns 使用主机 resolve.conf 并设置 dns 上游服务器地址

{% tabs %}
{% tab title="inventory/mycluster/group\_vars/all/all.yml" %}
```text
upstream_dns_servers:
- <ip>:<port>
```
{% endtab %}

{% tab title="inventory/k8s-cluster/k8s-cluster.yml" %}
```
resolvconf_mode: host_resolvconf
```
{% endtab %}
{% endtabs %}

2、更新 dns

```text
ansible-playbook -i inventory/mycluster/hosts.yaml --tags "apps,resolvconf" --
```

## 离线安装

在某些私有部署的场景下，无法连接互联网进行下载安装，这需要提前将安装所需的各种依赖资源下载到本地，方法如下：

{% embed url="https://github.com/kubernetes-sigs/kubespray/blob/master/docs/offline-environment.md" %}

