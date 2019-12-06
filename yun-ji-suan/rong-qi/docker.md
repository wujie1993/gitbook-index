# Docker

## 快速开始

### 安装

```text
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce
```

### 配置

```text
{
        "registry-mirrors": ["https://dockerhub.azk8s.cn"],
        "live-restore": true
}
```

### 启动

```text
systemctl enable docker
systemctl start docker
```

