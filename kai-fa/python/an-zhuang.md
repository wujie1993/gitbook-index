# 安装

## 手动安装

### 下载地址

{% embed url="https://www.python.org/downloads/source/" %}

### 安装步骤

以安装3.8.2版本为例，首先获取源代码

```text
wget https://www.python.org/ftp/python/3.8.2/Python-3.8.2.tar.xz
tar xvf Python-3.8.2.tar.xz && Python-3.8.2
```

安装依赖库

```text
yum install -y zlib zlib-devel
```

根据源码编译并安装

```text
./configure
make && make install
```

查看python安装位置

```text
$ ll /usr/local/bin/ | grep python3
lrwxrwxrwx. 1 root root        9 Mar  9 17:04 python3 -> python3.8
-rwxr-xr-x. 1 root root 17099048 Mar  9 17:04 python3.8
-rwxr-xr-x. 1 root root     3087 Mar  9 17:04 python3.8-config
lrwxrwxrwx. 1 root root       16 Mar  9 17:04 python3-config -> python3.8-config
```

## pyenv

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

