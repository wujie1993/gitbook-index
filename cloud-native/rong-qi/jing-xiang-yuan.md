# 镜像源

目前互联网中主要的几个镜像仓库在国内拉取镜像时速度很慢甚至是响应超时，以下列出了几个在国内可用的代理镜像仓库

| 目标 | 代理地址 |
| :--- | :--- |
| docker.io |  [dockerhub.azk8s.cn](http://mirror.azk8s.cn/help/docker-registry-proxy-cache.html) |
| gcr.io |  [gcr.azk8s.cn](http://mirror.azk8s.cn/help/gcr-proxy-cache.html) |
| quay.io |  [quay.azk8s.cn](http://mirror.azk8s.cn/help/quay-proxy-cache.html) |
| gcr.io | [gcr.mirrors.ustc.edu.cn](http://gcr.mirrors.ustc.edu.cn/) |

例子：拉取 gcr.io/google\_containers/kubernetes-dashboard-amd64:v1.7.0镜像

```text
docker pull gcr.azk8s.cn/google_containers/kubernetes-dashboard-amd64:v1.7.0
docker tag gcr.azk8s.cn/google_containers/kubernetes-dashboard-amd64:v1.7.0 gcr.io/google_containers/kubernetes-dashboard-amd64:v1.7.0
```

