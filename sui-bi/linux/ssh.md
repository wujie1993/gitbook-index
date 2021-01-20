# SSH

## 客户端

生成ssh密钥

```text
ssh-keygen -o -t rsa -b 4096 -C "email@example.com"
```

执行完成后会同时生成密钥与公钥，其中公钥可用于git服务端验证，通过以下命令获取公钥内容

```text
cat ~/.ssh/id_rsa.pub
```

**常用参数**

```text
-b 密钥长度，单位bit
-t 使用的加密算法，可选[ dsa | ecdsa | ed25519 | rsa ]
-C 备注信息，在git上使用需要添加邮箱地址
-f 输出路径，默认为~/.ssh/id_rsa
```

当需要同时连接多个git仓库，而使用的邮箱又不相同时，可以通过`~/.ssh/config`配置针对不同域名访问时所使用的策略

```text
Host github.com
    HostName github.com
    IdentityFile ~/.ssh/id_rsa
    Port 22
Host example.com
    IdentityFile ~/.ssh/example
```

## Q&A

### 如何配置免密ssh登录？

通过命令`ssh-copy-id {{ 主机名 }}`

或者

在`~/.ssh/authorized_keys`中存放着已授权的密钥，如果配置`host1`连接`host2`的免密码ssh登录，可以将`host1`上的公钥（默认：`~/.ssh/id_rsa.pub`）中的内容拷贝追加到`host2`上的`~/.ssh/authorized_keys`中

### 如何配置免密root登录？

```text
# visudo

// 追加以下内容
{{ 用户名 }} ALL=(ALL) NOPASSWD: ALL
```

### 如何配置ssh代理？

首先安装[nc](https://linux.die.net/man/1/nc)工具

```text
yum install nc -y
```

在`~/.ssh/config`中为需要代理的主机名配置ProxyCommand，其中-x参数指定的是sock5代理的地址

```text
Host github.com   
  IdentityFile ~/.ssh/id_rsa 
  ProxyCommand nc -x localhost:1080 %h %p
```

