# Harbor

## 项目地址

{% embed url="https://github.com/goharbor/harbor" %}

## 快速开始

1、下载 Harbor 离线安装包

```text
wget https://github.com/goharbor/harbor/releases/download/v2.2.0/harbor-offline-installer-v2.2.0.tgz && tar xvf harbor-offline-installer-v2.2.0.tgz && mv ./harbor /opt/harbor && cd /opt/harbor
```

2、配置 harbor.yml

```text
cp harbor.yml.tmpl harbor.yml
```

{% code title="harbor.yml" %}
```text
hostname: 10.14.31.129
external_url: http://10.14.31.129
```
{% endcode %}

3、安装启动 Harbor

```text
./prepare
./install.sh
```

4、浏览器访问 Harbor 地址 http://&lt;ip&gt;:80，使用 admin/Harbor12345 登录管理员账号

