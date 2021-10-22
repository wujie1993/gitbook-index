# Harbor

## 项目地址

{% embed url="https://github.com/goharbor/harbor" %}

## 快速开始

1、下载 Harbor 离线安装包

```
wget https://github.com/goharbor/harbor/releases/download/v2.2.0/harbor-offline-installer-v2.2.0.tgz
tar xvf harbor-offline-installer-v2.2.0.tgz
mv ./harbor /opt/harbor
cd /opt/harbor
```

2、配置 harbor.yml

```
cp harbor.yml.tmpl harbor.yml
```

{% code title="harbor.yml" %}
```
# 配置主机名
hostname: 10.14.31.129

# 配置对外宣告的地址
external_url: http://10.14.31.129

# 配置存储目录
data_volume: /data/harbor

# 注释 https 块
#https:
#  # https port for harbor, default is 443
#  port: 443
#  # The path of cert and key files for nginx
#  certificate: /your/certificate/path
#  private_key: /your/private/key/path
```
{% endcode %}

3、安装启动 Harbor

```
# 基于 harbor.yml 生成配置
./prepare
# 安装镜像并启动服务
./install.sh
```

4、浏览器访问 Harbor 地址 http://\<ip>:80，使用 admin/Harbor12345 登录管理员账号
