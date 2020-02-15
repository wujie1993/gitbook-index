# Git

### 初始化仓库

```text
git init
```

### 配置用户名和邮箱

```text
git config user.username "yourname"
git config user.email "your@email.com"
```

如果需要使配置生效，可附加参数`--global`

## 实用技巧

### github代理

git拉取推送代码的方式有http和ssh两种

http方式的代理可通过设置环境变量http\_proxy和https\_proxy指定。

```text
export http_proxy=http://{{ http代理服务地址 }}:{{ http代理服务端口 }}
export https_proxy=http://{{ http代理服务地址 }}:{{ http代理服务端口 }}
```

ssh方式的代理可通过配置ssh的代理命令指定

{% code title="~/.ssh/config" %}
```text
Host github.com
        User    git
        Hostname        github.com
        Port    22
        Proxycommand    /usr/bin/ncat --proxy 127.0.0.1:1081 --proxy-type socks5 %h %p
```
{% endcode %}

以上方式是直接修改系统的代理配置，也可通过git自身的配置修改代理，通过命令行

```text
git config --global http.proxy 'socks5://127.0.0.1:1081'
git config --global https.proxy 'socks5://127.0.0.1:1081'
```

或者修改配置文件

{% code title="~/.gitconfig" %}
```text
[http]
        proxy = socks5://127.0.0.1:1081
[https]
        proxy = socks5://127.0.0.1:1081
```
{% endcode %}

### 将多次提交进行合并

{% embed url="https://segmentfault.com/a/1190000007748862" %}

### 撤销上回的提交

1. `git reset commitId`，\(注：不要带--hard\)到上个版本  
2. `git stash`，暂存修改  
3. `git push --force`, 强制push,远程的最新的一次commit被删除  
4. `git stash pop`，释放暂存的修改，开始修改代码  
5. `git add .` -&gt; `git commit -m "massage"` -&gt; `git push`

如果是仅修改commit描述

1. `git commit --amend`\(注：修改完成后`Esc+wq+Enter`保存退出\)  
2. `git push --force`

