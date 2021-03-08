---
description: 部署生产环境就绪的kubernetes集群
---

# kubespray

## 项目地址

{% embed url="https://github.com/kubernetes-sigs/kubespray" %}

## 快速开始

前置动作：

* 关闭防火墙或配置服务器间的互信任策略
* 系统时钟同步

1、下载kubespray

```text
git@github.com:kubernetes-sigs/kubespray.git && cd kubespray
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

拉取gcr.io的镜像可以配置国内镜像源gcr\_image\_repo: registry.aliyuncs.com
{% endhint %}

## 离线安装

在某些私有部署的场景下，无法连接互联网进行下载安装，这需要提前将安装所需的各种依赖资源下载到本地，方法如下：

{% embed url="https://github.com/kubernetes-sigs/kubespray/blob/master/docs/offline-environment.md" %}

