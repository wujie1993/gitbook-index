# 原理

### 如何生成各种步长精度的查询结果

{% embed url="https://cloud.tencent.com/developer/article/1382875" %}

## 本地存储

prometheus的数据分布在一个个的block目录中，每个block目录中存储时长2小时的数据。block中的数据经过压缩分为一个个的chunk，每个chunk默认大小为512MB。block中的index存储该block中的指标标签和各种索引，meta.json存储block的元数据信息，tombstones存储已经被软删除的数据偏移位置

