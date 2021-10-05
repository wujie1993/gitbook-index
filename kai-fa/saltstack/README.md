# SaltStack

### 官方文档

{% embed url="https://docs.saltproject.io/en/latest/contents.html" %}

### 项目地址

{% embed url="https://github.com/saltstack/salt" %}

### 快速开始

1、配置安装源

```text
rpm --import https://repo.saltproject.io/py3/redhat/7/x86_64/latest/SALTSTACK-GPG-KEY.pub
curl -fsSL https://repo.saltproject.io/py3/redhat/7/x86_64/latest.repo | sudo tee /etc/yum.repos.d/salt.repo
```

2、安装 salt-master

```text
yum install salt-master
```

3、启动 salt-master 并配置开机自启动

```text
systemctl start salt-master
systemctl enable salt-master
```

4、安装 salt-minion

```text
yum install salt-minion
```

5、配置 salt-minion

{% code title="/etc/salt/minion" %}
```text
# master 设置 salt-master 节点地址
master: salt-master
# id 设置当前 salt-minion 节点的标识 id
id: salt-minion01
```
{% endcode %}

6、启动 salt-minion 并设置开机自启动

```text
systemctl start salt-minion
systemctl enable salt-minion
```

7、在 salt-master 节点上查看等待连接的 salt-minion 密钥，并允许连接

```text
salt-key -L
salt-key -a salt-minion01 -y
```

{% hint style="info" %}
可使用 -d 参数替换 -a 参数移除 salt-minion 密钥
{% endhint %}

8、 测试连接

```text
salt "*" test.ping
```

### 参考资料

{% embed url="https://www.cnblogs.com/yanjieli/p/10864648.html?page=4" %}



