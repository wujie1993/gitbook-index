# Openssl

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

