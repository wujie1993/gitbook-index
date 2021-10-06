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
pip3 install PyOpenSSL
```

3、生成 tls 证书

```text
salt-call --local tls.create_self_signed_cert
```

### 参考文档

{% embed url="https://www.cnblogs.com/yanjieli/p/10916198.html" %}





