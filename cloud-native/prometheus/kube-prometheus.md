# kube-prometheus

## 项目地址

{% embed url="https://github.com/prometheus-operator/kube-prometheus" %}

## 快速开始

1、下载 kube-prometheus

```text
mkdir -p /opt/k8s/prometheus && cd /opt/k8s/prometheus
git clone https://github.com/prometheus-operator/kube-prometheus.git && cd kube-prometheus && git checkout v0.7.0
```

2、安装 kube-prometheus

```text
kubectl create -f manifests/setup
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done
kubectl create -f manifests/
```

3、暴露 prometheus 与 grafana 对外服务端口

```text
kubectl patch svc -n monitoring grafana --type=json -p='[{"op": "add", "path": "/spec/ports/0/nodePort", "value":30002}, {"op": "replace", "path": "/spec/type", "value": "NodePort"}]'
kubectl patch svc -n monitoring prometheus-k8s --type=json -p='[{"op": "add", "path": "/spec/ports/0/nodePort", "value": 30003}, {"op": "replace", "path": "/spec/type", "value": "NodePort"}]'
```

4、访问 http://:30002，使用 admin/admin 登录

