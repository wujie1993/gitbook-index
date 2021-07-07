# Mysql

### 参考文章

{% embed url="https://xie.infoq.cn/article/4e7de8ce0607bfbe24a5c2e16" %}

### 使用

例子：创建只读用户

```text
-- 创建用户   用户名：readonly  密码：123456 可自行更改 --
CREATE USER 'readonly'@'%' IDENTIFIED BY '123456';

-- 赋予用户只读权限  库名：mydb  用户：readonly --
GRANT SELECT ON mydb.* TO 'readonly'@'%';

-- 刷新权限  --
FLUSH PRIVILEGES;
```

