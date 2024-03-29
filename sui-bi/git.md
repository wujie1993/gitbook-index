# Git

## 安装

{% embed url="https://cloud.tencent.com/developer/article/1590046" %}

## 使用

### 初始化仓库

```
git init
```

### 配置作者信息

在项目提交前需要配置用户名和邮箱，在许多开源项目中还要求添加签名

```
git config user.username {{ 用户名 }}
git config user.email {{ 邮箱 }}
```

或者修改项目中的git配置文件

{% code title=".git/config" %}
```
[user]
        name = {{ 用户名 }}
        email = {{ 邮箱 }}
```
{% endcode %}

如果需要使配置全局生效，可附加参数`--global`

在提交commit后如果发现其中的作者信息没有更新，可以使用下述命令重置作者信息并重新提交

```
 git commit -s --amend --reset-author
```

## 子模块

例子：添加子模块

```
git submodule add <项目地址> <目标路径>
```

## 实用技巧

### 大文件存储git-lfs

{% embed url="https://www.jianshu.com/p/493b81544f80" %}

### github代理

git拉取推送代码的方式有http和ssh两种

#### http代理

http方式的代理可通过设置环境变量http\_proxy和https\_proxy指定。

```
export http_proxy=http://{{ http代理服务地址 }}:{{ http代理服务端口 }}
export https_proxy=http://{{ http代理服务地址 }}:{{ http代理服务端口 }}
```

以上方式是直接修改系统的代理配置，也可通过git自身的配置修改代理，通过命令行

```
git config --global http.proxy 'socks5://127.0.0.1:1081'
git config --global https.proxy 'socks5://127.0.0.1:1081'
```

或者修改配置文件

{% code title="~/.gitconfig" %}
```
[http]
        proxy = socks5://127.0.0.1:1081
[https]
        proxy = socks5://127.0.0.1:1081
```
{% endcode %}

#### ssh代理

首先安装[nc](https://linux.die.net/man/1/nc)工具

```
yum install nc -y
```

在`~/.ssh/config`中为需要代理的主机名配置ProxyCommand

{% code title="~/.ssh/config" %}
```
Host github.com
        User    git
        Hostname        github.com
        Port    22
        Proxycommand    /usr/bin/ncat --proxy 127.0.0.1:1081 --proxy-type socks5 %h %p
```
{% endcode %}

### 将多次提交进行合并

{% embed url="https://segmentfault.com/a/1190000007748862" %}

### 撤销已提交的改动

1\. `git reset commitId`，(注：不要带--hard)到上个版本\
2\. `git stash`，暂存修改\
3\. `git push --force`, 强制push,远程的最新的一次commit被删除\
4\. `git stash pop`，释放暂存的修改，开始修改代码\
5\. `git add .` -> `git commit -m "massage"` -> `git push`

如果是仅修改commit描述

1\. `git commit --amend`(注：修改完成后`Esc+wq+Enter`保存退出)\
2\. `git push --force`

### 撤销未add的改动

```
git checkout -- {{ path }}
```

### 撤销已add但未commit的改动

```
git reset {{ commit_id or branch or path}}
```
