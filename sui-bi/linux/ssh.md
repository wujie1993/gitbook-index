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

## 服务端
