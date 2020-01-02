# Q&A

## 系统负载过高

排查是否存在label值维度过大的指标

```text
count by (__name__) ({__name__=~".+"}) > 10000
```

排查是否使用了nfs文件系统（建议使用ssd）

排查job的采集间隔是否过小（一般15或30秒）

排查是否采集了多余的targets

## 时区修复

```text
cp /usr/share/zoneinfo/PRC ./

cat >> Dockerfile_prometheus << EOF

FROM quay.io/prometheus/prometheus:v2.13.1
ADD ./PRC /etc/localtime

EOF

docker build . -f Dockerfile_prometheus -t prometheus:v2.13.1-fix-timezone
```

