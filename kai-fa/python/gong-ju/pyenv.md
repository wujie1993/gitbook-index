---
description: 便携式python沙盒执行环境
---

# pyenv

pyenv用于python的版本管理，每个版本的python都是用独立的存储区域，并且可依据需要自由切换和组合不同的python版本。

### 项目地址

{% embed url="https://github.com/pyenv/pyenv\#installation" %}

### 快速开始

安装git用于拉取代码

```text
yum install -y git
```

安装pyenv

```text
curl https://pyenv.run | bash
```

将以下内容追加到~/.bashrc中，使每次登录时自动生效

```text
export PATH="/root/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

以安装python 3.8.0为例，先使用yum安装编译所需的依赖库，在通过pyenv安装python 3.8.0

```text
yum install -y  zlib-devel openssl-devel bzip2-devel readline-devel sqlite-devel libffi-devel
pyenv install 3.8.0
```

安装后的python存储于/root/.pyenv/versions/3.8.0/bin中

