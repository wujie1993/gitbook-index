---
description: python包管理工具
---

# pip

### 快速上手

1、安装 pip 工具

{% tabs %}
{% tab title="Ubuntu" %}
```text
apt install python3-pip
```
{% endtab %}
{% endtabs %}

2、更新 pip 工具

```text
pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U
```

3、配置 pip 源

```text
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

### 使用

通过requirements.txt文件安装python依赖包

```text
pip3 install -r requirements.txt
```



