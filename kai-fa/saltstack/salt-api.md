---
description: saltstack接口服务
---

# salt-api

### 安装

1、安装 salt-api

```text
yum install salt-api
```

2、安装 python 依赖

```text
pip3 install PyOpenSSL
```

3、生成 tls 证书

```text
salt-call --local tls.create_self_signed_cert
```

4、开启配置文件子目录

{% code title="/etc/salt/master" %}
```text
default_include: master.d/*.conf
```
{% endcode %}

5、配置 salt-api

{% code title="/etc/salt/master.d/api.conf" %}
```text
rest_cherrypy:
  host: 0.0.0.0
  port: 8000
  ssl_crt: /etc/pki/tls/certs/localhost.crt
  ssl_key: /etc/pki/tls/certs/localhost.key
```
{% endcode %}

6、创建 salt-api 认证账号

```text
useradd -M -s /sbin/nologin saltapi
echo 'saltapi' | passwd --stdin saltapi
```

7、配置 salt-master 使用 salt-api 认证

{% code title="/etc/salt/master.d/auth.conf" %}
```text
external_auth:
  pam:
    saltapi:
      - .*
      - '@wheel'
      - '@runner'
      - '@jobs'
```
{% endcode %}

8、重启服务

```text
systemctl restart salt-master
systemctl start salt-api
systemctl enable salt-api
```

9、本地登录测试

```text
curl -sSk https://localhost:8000/login \
-H 'Accept: application/x-yaml' \
-d username=saltapi \
-d password=saltapi \
-d eauth=pam
```

### 参考文档

{% embed url="https://www.cnblogs.com/yanjieli/p/10916198.html" %}





