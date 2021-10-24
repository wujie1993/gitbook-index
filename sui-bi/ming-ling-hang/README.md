# 命令行工具

## awk

例子：打印输出内容的第n列

```
$ awk '{print $n}'
```

例子： 将csv格式的文件按字段进行分组切割

```
$ cat << EOF > staff.csv
US Gavo 35
US Jane 21
US Bill 25
China Jimmy 42
EOF

$ awk '{print > $1".csv"}' staff.csv

$ ls
China.csv  staff.csv  US.csv

$ cat China.csv
China Jimmy 42

$ cat US.csv
US Gavo 35
US Jane 21
US Bill 25
```

## grep

例子：打印带有Ready关键字的行

```
$ cat << EOF > text
I'm Ready.
But it's too late.
EOF

$ cat text | grep "Ready"
I'm Ready.
```

**可选参数**

```
-i    忽略大小写
-n    显示行号
–color    高亮关键字，centos7默认已经高亮
-c    统计符合条件的行数
-o    只打印关键字，每个被匹配的关键字单独显示一行
-B    同时显示之前的行，后面必须有数字，如 -B2
-A    同时显示之后的行
-w    只匹配独立单词，也就是精确匹配
-v    反向查找
-e    同时匹配多个目标
-q    静默模式，只关心有没有匹配到，不关心内容
-E    可以使用扩展正则，相当于egrep
-P    使用兼容perl的正则
-r    递归
-I    忽略二进制文件（默认情况下grep不会忽略二进制文件）
```

例子：过滤空白行

```
$ grep -Ev "^$" test.txt
```

## wc

例子：统计文件中文本行数

```
$ cat << EOF > text
I'm Ready.
But it's too late.
EOF

$ cat text | wc -l
2
```

## shell

例子：删除命名空间openshift-monitoring中的所有PrometheusRule

```
$ oc get PrometheusRule --all-namespaces|grep -v NAMESPACE|awk '{print $2}'|while read name; do oc delete prometheusrule $name -n openshift-monitoring; done
```

## sed

例子：替换文件内容

```
$ sed -i 's/替换前的内容/替换后的内容/g' file.txt
```

例子：文件内容追加

```
$ sed -i '1i 在第一行头部追加的内容' file.txt
$ sed -i '1a 在第一行尾部追加的内容' file.txt
$ sed -i '$i 在最后一行头部追加的内容' file.txt
$ sed -i '$i 在最后一行尾部追加的内容' file.txt
```

例子：替换匹配的行

```
$ sed -i '/ swap / s/^/#/' /etc/fstab
```

例子：删除匹配的行

```
# 移除/etc/profile中golang环境变量
$ sed -i '/export PATH=\$PATH:\/usr\/local\/go\/bin:\/root\/go\/bin/d' /etc/profile
$ sed -i '/export GOPATH=\/root\/go/d' /etc/profile

# 删除/etc/fstab中的注释行
$ sed -i 's/^#/d' /etc/profile
```

### 快速指令

例子：根据进程查询其监听端口

```
netstat -anp|grep $(ps -ef|grep <进程名>|grep -v "grep"|awk '{print $2}')|grep LISTEN|awk '{print $4}'
```
