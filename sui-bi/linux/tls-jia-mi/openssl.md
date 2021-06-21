# Openssl

## 安装

```text
yum install openssl -y
```

## 查看证书

查看证书内容

```text
openssl x509 -in {{ 证书路径 }} -noout -text
```

查看证书序列号

```text
openssl x509 -in {{ 证书路径 }} -noout -serial
```

查看证书拥有者

```text
openssl x509 -in {{ 证书路径 }} -noout -subject
```

## 生成证书

例子：生成单向加密证书

```text
# 生成私钥
openssl genrsa -out server.key 1024
# 生成证书
openssl req -new -x509 -days 3650 -key server.key -out server.crt -subj "/C=CN/ST=yourprovince/L=yourcity/O=yourorg/CN=yourdomain"
```

{% embed url="https://blog.csdn.net/u013958257/article/details/107326771" %}



## 参考资料



