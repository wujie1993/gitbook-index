# 账号

### 免密码认证

编辑/etc/sudoers

```text
# 使指定用户在使用sudo运行命令时无需输入密码
{{ 用户名 }} ALL=(ALL:ALL) NOPASSWD:ALL
```

### 使用sudo su时缺少/etc/profile中的环境变量

在~/.bachrc中添加source /etc/profile语句

