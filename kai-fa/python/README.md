# Python

Python是一种支持动态类型的高级脚本语言，由 [Guido van Rossum](https://en.wikipedia.org/wiki/Guido_van_Rossum)发布于1991，它支持多种编程规范，包括过程，面向对象和函数式编程，目前广泛应用于运维开发和人工智能领域。‌

在CentOS7中默认安装python 2.7，在CentOS8中则默认安装python 3。‌

## 官方网站

{% embed url="https://www.python.org/" %}

## 项目地址

{% embed url="https://github.com/python/cpython" %}

## 手动安装

{% embed url="https://www.python.org/downloads/source/" caption="下载地址" %}

以安装3.8.2版本为例，首先获取源代码

```text
wget https://www.python.org/ftp/python/3.8.2/Python-3.8.2.tar.xz
tar xvf Python-3.8.2.tar.xz && cd Python-3.8.2
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

## 参考资料

* [通过python实现所有算法](https://github.com/TheAlgorithms/Python)

