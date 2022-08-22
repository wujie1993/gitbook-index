# Kubeflow

### 官方网站

{% embed url="https://www.kubeflow.org/" %}

### 项目地址

{% embed url="https://github.com/kubeflow/kubeflow" %}

### 快速开始

1、环境准备

* k8s v1.22.8 (配置默认StorageClass)
* kustomize 3.2.0
* kubectl v1.22.8

2、拉取部署脚本

```bash
mkdir /opt/kubeflow && cd /opt/kubeflow
wget https://github.com/kubeflow/manifests/archive/refs/tags/v1.6.0-rc.1.tar.gz
tar -zxvf v1.6.0-rc.1.tar.gz && cd v1.6.0-rc.1
```

由于kubeflow部署过程中需要使用到外网镜像，建议先将使用到的镜像拉取到本地仓库中，然后修改kustomization.yaml文件，配置images项修改镜像名

3、一键部署

```
while ! kustomize build example | kubectl apply -f -; do echo "Retrying to apply resources"; sleep 10; done
```
