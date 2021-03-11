# 镜像仓库

目前互联网中主要的几个镜像仓库在国内拉取镜像时速度很慢甚至是响应超时，以下列出了几个在国内可用的代理镜像仓库

| 目标 | 代理地址 |
| :--- | :--- |
| docker.io | hub-mirror.c.163.com,docker.mirrors.ustc.edu.cn |
| quay.io | quay.mirrors.ustc.edu.cn |
| gcr.io | gcr.mirrors.ustc.edu.cn,registry.aliyuncs.com |
|  k8s.gcr.io | registry.aliyuncs.com/google\_containers |

例子：拉取 gcr.io镜像

```text
docker pull gcr.mirrors.ustc.edu.cn/google_containers/kubernetes-dashboard-amd64:v1.7.0
docker tag gcr.mirrors.ustc.edu.cn/google_containers/kubernetes-dashboard-amd64:v1.7.0 gcr.io/google_containers/kubernetes-dashboard-amd64:v1.7.0
docker rmi gcr.mirrors.ustc.edu.cn/google_containers/kubernetes-dashboard-amd64:v1.7.0
```

