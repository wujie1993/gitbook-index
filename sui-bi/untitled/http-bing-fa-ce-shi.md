# apache bench

apache bench，简称为ab

ab是apache自带的压力测试工具。ab非常实用，它不仅可以对apache服务器进行网站访问压力测试，也可以对或其它类型的服务器进行压力测试。比如nginx、tomcat、IIS等。

### 安装

```text
yum install httpd -y
```

### 测试

```text
ab -n 100 -c 10 https://www.baidu.com/
```

**参数解析**

* **-n 请求数**
* **-c 并发数**

