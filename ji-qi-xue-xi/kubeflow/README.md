# Kubeflow

### 官方网站

{% embed url="https://www.kubeflow.org/" %}

### 项目地址

{% embed url="https://github.com/kubeflow/kubeflow" %}

### 快速开始

#### 本地部署

1、安装 VSCode

2、安装 Jupyter 插件

3、在创建 .ipynb 文件后，会有弹窗提示安装相应依赖，点击确认安装即可

#### k8s部署

1、环境准备

* k8s v1.22.8 (配置默认StorageClass)
* kustomize 3.2.0
* kubectl v1.22.8

2、拉取部署脚本

```bash
mkdir /opt/kubeflow && cd /opt/kubeflow
wget https://github.com/kubeflow/manifests/archive/refs/tags/v1.6.1.tar.gz
tar -zxvf v1.6.1.tar.gz && cd v1.6.1
```

由于kubeflow部署过程中需要使用到外网镜像，建议先将使用到的镜像拉取到本地仓库中，然后修改kustomization.yaml 文件，配置 images 项修改镜像名

> 同步外网镜像可参考 [https://github.com/wujie1993/mirrors](https://github.com/wujie1993/mirrors)

3、一键部署

```
while ! kustomize build example | kubectl apply -f -; do echo "Retrying to apply resources"; sleep 10; done
```
