# Journalctl

例子：查询某个时段的kubelet日志

```text
journalctl -xe --since="2019-11-09 17:38:09" --until="2019-11-10" -u kubelet
```



