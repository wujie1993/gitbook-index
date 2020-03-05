# 命令行工具

## awk

例子：打印输出内容的第n列

```text
awk '{print $n}'
```

例子： 将csv格式的文件进行分割

```text
cat << EOF >staff.csv
US Gavo 35
US Jane 21
US Bill 25
China Jimmy 42
EOF


# 以国家名称进行分组
awk '{print > $1".csv"}' staff.csv
```

## grep

例子：打印具带有关键字的行

```text
grep Ready 
```

## wc

例子：统计行数

```text
wc -l
```

## shell

例子：删除命名空间openshift-monitoring中的所有PrometheusRule

```text
oc get PrometheusRule --all-namespaces|grep -v NAMESPACE|awk '{print $2}'|while read name; do oc delete prometheusrule $name -n openshift-monitoring; done
```

## sed

例子：替换文件内容

```text
sed -i 's/替换前的内容/替换后的内容/g' file.txt
```

例子：文件内容追加

```text
sed -i '1i 在第一行头部追加的内容' file.txt
sed -i '1a 在第一行尾部追加的内容' file.txt
sed -i '$i 在最后一行头部追加的内容' file.txt
sed -i '$i 在最后一行尾部追加的内容' file.txt
```

例子：替换匹配的行

```text
sed -i '/ swap / s/^/#/' /etc/fstab
```

例子：删除匹配的行

```text
# 移除/etc/profile中golang环境变量
sed -i '/export PATH=\$PATH:\/usr\/local\/go\/bin:\/root\/go\/bin/d' /etc/profile
sed -i '/export GOPATH=\/root\/go/d' /etc/profile

# 删除/etc/fstab中的注释行
sed -i 's/^#/d' /etc/profile
```

