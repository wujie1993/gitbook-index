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

prometheus默认使用格林威治时间，发送到alertmanager的告警通知请求中携带了触发时间，alertmanager会将该时间作为告警触发的时间，为了避免告警时间出现偏差，需要将prometheus的时区改为当前所使用的时区

```text
cp /usr/share/zoneinfo/PRC ./

cat >> Dockerfile_prometheus << EOF

FROM quay.io/prometheus/prometheus:v2.13.1
ADD ./PRC /etc/localtime

EOF

docker build . -f Dockerfile_prometheus -t prometheus:v2.13.1-fix-timezone
```

