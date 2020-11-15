# Ansible

### 开发自定义模块

{% embed url="https://docs.ansible.com/ansible/latest/dev\_guide/developing\_modules\_general.html" %}

### 任务运行时间统计

配置ansible.cfg

```text
[defaults]
callback_whitelist = profile_tasks
```

### 错误日志格式化

配置ansible.cfg

```text
[defaults]
stdout_callback = yaml
```

可选值：

### 加速器

安装mitogen加速模块

```text
pip install mitogen
```

配置ansible.cfg

```text
[defaults]
strategy = mitogen_linear
strategy_plugins = /usr/lib/python2.7/site-packages/ansible_mitogen/plugins/strategy
```

### Q&A

**Q: 如何获取ansible主控机器地址**

```text
- debug: var="{{ ansible_env['SSH_CLIENT'].split() | first }}"
```

**Q: playbook运行卡在gather\_facts**

将gather\_facts信息采集改为最少

```text
[default]
gather_subset = min
```

