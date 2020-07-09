# WSL2

## Q&A

### 如何配置wsl 2资源限制

{% code title="%UserProfile%\\.wslconfig" %}
```text
[wsl2]
memory=6GB
swap=0
localhostForwarding=true
```
{% endcode %}

{% embed url="https://docs.microsoft.com/en-us/windows/wsl/release-notes\#build-18945" %}

### 重启wsl 2

打开 任务管理器 -&gt; 服务，重启LxssManager服务

### 如何通过ssh连接wsl 2

重装openssh-server

```bash
sudo apt-get remove openssh-server
sudo apt-get install openssh-server
```

配置sshd

{% code title="/etc/ssh/sshd\_config" %}
```text
# 修改端口不与宿主机冲突
Port 2222
# 允许使用密码认证方式登录
PasswordAuthentication yes
```
{% endcode %}

重启sshd

```bash
 sudo service ssh --full-restart
```

在windows宿主机上测试连接

```bash
ssh 127.0.0.1 2222
```

