# 部署

## 单机minikube

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
yum install -y kubectl
```

### 安装minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-1.3.1.rpm \
 && sudo rpm -ivh minikube-1.3.1.rpm
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



