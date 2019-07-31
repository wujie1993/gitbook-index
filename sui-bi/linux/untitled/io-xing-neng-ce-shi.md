# IO性能测试

对于IO的性能测试比较常用的工具有大部分linux系统自带的dd工具

## dd

在使用dd工具进行性能测试时，需要附加参数oflag=dsync或conv=fdatasync，否则测试结果会因为缓存机制而出现误差

### 读取测试

例子：使用dd进行硬盘写入测试

```text
dd if=/dev/zero of=/tmp/test1.img bs=1G count=1 oflag=dsync
```

或

```text
dd if=/dev/zero of=/tmp/testALT.img bs=1G count=1 conv=fdatasync
```

{% hint style="info" %}
/dev/zero是一个虚拟的设备，从/dev/zero设备中可以不断地读取到0x00值，其读取速度远远高于硬盘，因此作为数据的读取来源
{% endhint %}

### 写入测试

例子：使用dd进行硬盘读取测试

```text
dd if=/tmp/test1.img of=/dev/null bs=1G count=1 oflag=dsync
```

或

```text
dd if=/dev/zero of=/dev/null bs=1G count=1 conv=fdatasync
```

{% hint style="info" %}
/dev/null是一个虚拟的设备，对于/dev/null的所有数据写入都会被丢弃，其写入速度远远高于硬盘，因此作为数据的写入目标
{% endhint %}

### 参数解析

* if：数据的读取来源
* of：数据的写入目标
* bs：数据分块大小
* count：数据分块总数

