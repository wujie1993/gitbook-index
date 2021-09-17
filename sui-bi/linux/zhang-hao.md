# 账号

### 添加用户

```text
useradd {{ 用户名 }}
```

### 添加用户组

```text
groupadd -g {{ 用户组id }} {{ 用户组 }}
```

### 为用户添加用户组

```text
usermod -a -G {{ 用户组 }} {{ 用户名 }}
```

{% hint style="info" %}
`-a`表示为只添加用户组，不加`-a`时会覆盖掉其他用户组
{% endhint %}

### 免密码认证

编辑/etc/sudoers

```text
# 使指定用户在使用sudo运行命令时无需输入密码
{{ 用户名 }} ALL=(ALL:ALL) NOPASSWD:ALL
```

### 使用sudo su时缺少/etc/profile中的环境变量

在~/.bachrc中添加source /etc/profile语句

