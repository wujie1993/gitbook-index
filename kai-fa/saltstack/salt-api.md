---
description: saltstack接口服务
---

# salt-api

### 安装

1、安装并启动 salt-api

```text
yum install salt-api
systemctl start salt-api
systemctl enable salt-api
```

2、安装 python 依赖

```text
pip install PyOpenSSL
```

3、生成 tls 证书

