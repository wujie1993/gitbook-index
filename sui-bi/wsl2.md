# WSL2

## Q&A

### 如何配置wsl2资源限制

{% code title="%UserProfile%\\.wslconfig" %}
```text
[wsl2]
memory=6GB
swap=0
localhostForwarding=true
```
{% endcode %}

{% embed url="https://docs.microsoft.com/en-us/windows/wsl/release-notes\#build-18945" %}

### 如何重启wsl2

打开 任务管理器 -&gt; 服务，重启LxssManager服务

