# prometheus-operator

## 项目地址

{% embed url="https://github.com/coreos/prometheus-operator" %}

## 快速开始 <a id="quickstart"></a>

1、创建命名空间

```text
kubectl create ns kube-monitoring
```

2、安装operator

```text
wget https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.46.0/bundle.yaml
sed -i 's/namespace: default/namespace: kube-monitoring/g' bundle.yaml
kubectl apply -f bundle.yaml
```



