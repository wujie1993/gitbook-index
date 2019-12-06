# 部署

## Minikube

环境准备：CentOS 7/8

minikube版本：1.3.1

### 安装kubectl

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubectl-1.15.5
```

### 安装minikube

```bash
yum install -y https://storage.googleapis.com/minikube/releases/latest/minikube-1.3.1.rpm
```

### 关闭防火墙

```bash
systemctl stop firewalld
systemctl disable firewalld
```

### 启动minikube

```text
minikube start --vm-driver=none --kubernetes-version v1.15.5
```

### 启动dashboard

```bash
kubectl create clusterrolebinding add-on-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:default
minikube dashboard --url=true
```

得到如下输出，dashboard侦听在localhost地址上

```text
* Verifying dashboard health ...
* Launching proxy ...
* Verifying proxy health ...
http://127.0.0.1:35951/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/
```

如果需要通过外部访问，按`CTRL+C`中断，并使用以下命令以NodePort方式暴露服务

```text
kubectl delete svc kubernetes-dashboard -n kube-system
kubectl expose deployment kubernetes-dashboard --type=NodePort --port=80 -n kube-system
```

### 在CentOS8中运行

默认情况下minikube使用iptables模式启动kube-proxy，在centos8中使用的是nftables，与kube-proxy使用的iptables指令不兼容，因而无法写入规则并导致ip地址转发不生效，解决这个问题的一种途径是以IPVS方式启动kube-proxy

1. 编辑kube-proxy配置

`kubectl edit configmap kube-proxy -n kube-system`

2. 修改`mode: 'ipvs'`

3. 加载内核ipvs模块

```text
modprobe ip_vs_rr
modprobe ip_vs_sh
modprobe ip_vs_wrr
modprobe ip_vs
```

{% hint style="info" %}
由于kube-proxy中使用的iptables指令与CentOS8不兼容，因此networkpolicy不会生效
{% endhint %}

