# 集群部署

## 脚本地址

{% embed url="https://github.com/openshift/openshift-ansible" %}

## 预配置

### DNS

在每台节点上，安装dnsmasq

{% code title="" %}
```text
yum install -y dnsmasq
```
{% endcode %}

添加Router域名解析策略

{% code title="/etc/dnsmasq.d/external-hosts.conf" %}
```text
address=/.apps.oc.local/192.168.149.129
address=/.apps.oc.local/192.168.149.130
```
{% endcode %}

启动dnsmasq并设置开机自启动

```text
systemctl start dnsmasq
systemctl enable dnsmasq
```

## 快速安装

规划安装3台节点，一个主节点和三个计算节点

| 节点名称 | IP地址 | ansible | master | node | infra | compute |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| okd-0 | 192.168.149.129 | yes | yes | yes | yes | no |
| okd-1 | 192.168.149.130 | no | no | yes | yes | no |
| okd-2 | 192.168.149.131 | no | no | yes | no | yes |

首先修改每个节点的主机名

{% tabs %}
{% tab title="okd-0:~/ \#" %}
```text
hostnamectl set-hostname okd-0
hostname okd-0
```
{% endtab %}

{% tab title="okd-1:~/ \#" %}
```
hostnamectl set-hostname okd-1
hostname okd-1
```
{% endtab %}

{% tab title="okd-2:~/ \#" %}
```
hostnamectl set-hostname okd-2
hostname okd-2
```
{% endtab %}
{% endtabs %}

配置hosts名称解析，在每个节点的/etc/hosts文件中追加以下内容

{% code title="okd-X:/etc/hosts" %}
```text
192.168.149.129 okd-0
192.168.149.130 okd-1
192.168.149.131 okd-2
```
{% endcode %}

以okd-0为集群引导节点，配置到所有节点的ssh免密码登录

```text
ssh-keygen
ssh-copy-id okd-0
ssh-copy-id okd-1
ssh-copy-id okd-2
```

为每个节点安装并更新必要的依赖软件和docker

{% code title="okd-X:~/ \#" %}
```bash
yum install -y wget git net-tools bind-utils yum-utils iptables-services bridge-utils bash-completion kexec-tools sos psacct
yum update -y
reboot
yum install -y docker
systemctl start docker
systemctl enable docker
```
{% endcode %}

在master节点安装openjdk与python-passlib

{% code title="okd-0:/root/Downloads/openshift-ansible-openshift-ansible-3.9.99-1/ \#" %}
```bash
yum install -y java-1.8.0-openjdk-headless python-passlib
```
{% endcode %}

在引导节点上安装ansible

{% code title="okd-0:/root/Downloads/openshift-ansible-openshift-ansible-3.9.99-1/ \#" %}
```bash
yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sed -i -e "s/^enabled=1/enabled=0/" /etc/yum.repos.d/epel.repo
yum -y --enablerepo=epel install ansible pyOpenSSL
```
{% endcode %}

由于openshift-ansible v3.9不支持ansible&gt;=2.8的版本，建议使用以下命令安装

```text
yum install https://releases.ansible.com/ansible/rpm/release/epel-7-x86_64/ansible-2.7.9-1.el7.ans.noarch.rpm pyOpenSSL -y
```

下载安装脚本

{% code title="okd-0:/root/Downloads/ \#" %}
```bash
wget https://github.com/openshift/openshift-ansible/archive/openshift-ansible-3.9.99-1.tar.gz
tar xvf openshift-ansible-3.9.99-1.tar.gz && cd openshift-ansible-openshift-ansible-3.9.99-1
```
{% endcode %}

配置inventory

{% code title="okd-0:/etc/ansible/hosts" %}
```text
[OSEv3:children]
masters
nodes
etcd

[OSEv3:vars]
openshift_deployment_type=origin
openshift_release=3.9

osm_cluster_network_cidr=10.128.0.0/14
openshift_portal_net=172.30.0.0/16
osm_host_subnet_length=9
openshift_disable_check=disk_availability,memory_availability

# 配置多租户网络隔离
os_sdn_network_plugin_name='redhat/openshift-ovs-multitenant'
# Router服务的默认域名后缀
openshift_master_default_subdomain=apps.oc.local
# 配置docker日志的滚动清理策略和非加密镜像仓库地址
openshift_docker_options='--registry-mirror=https://53mhb806.mirror.aliyuncs.com --log-driver json-file --insecure-registry=172.30.0.0/16 --log-opt max-size=1M --log-opt max-file=3'

# 安装Hawkular,启用metrics
openshift_metrics_install_metrics=true
openshift_metrics_hawkular_hostname=hawkular-metrics.oc.local
openshift_metrics_image_version=v3.9

# 配置认证方式
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider', 'filename': '/etc/origin/master/htpasswd'}]

[masters]
okd-0

[etcd]
okd-0
okd-1
okd-2

[nodes]
okd-0 openshift_schedulable=true openshift_node_labels="{'region': 'infra', 'zone': 'default'}"
okd-1 openshift_schedulable=true openshift_node_labels="{'region': 'infra', 'zone': 'default'}"
okd-2 openshift_schedulable=true openshift_node_labels="{'zone': 'default'}"
```
{% endcode %}

配置node域名解析

```bash
ansible nodes -m shell -a "mkdir -p /etc/origin/node"
ansible nodes -m shell -a "echo 'nameserver 8.8.8.8' > /etc/origin/node/resolv.conf"
```

安装预检查

{% code title="okd-0:/root/Downloads/openshift-ansible-openshift-ansible-3.9.99-1/ \#" %}
```bash
ansible-playbook playbooks/prerequisites.yml
```
{% endcode %}

安装集群

{% code title="okd-0:/root/Downloads/openshift-ansible-openshift-ansible-3.9.99-1/ \#" %}
```bash
ansible-playbook playbooks/deploy_cluster.yml
```
{% endcode %}

{% hint style="info" %}
必须保证安装脚本完全执行通过，如果出现错误可以在排查完错误后重复执行
{% endhint %}

## 多主高可用

主节点采用keepalived绑定虚拟地址192.168.149.135做为master入口地址

规划安装3台节点，三个都作为主节点和计算节点

| 节点名称 | IP地址 | ansible | master | node | infra | compute |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| okd-0 | 192.168.149.129 | yes | yes | yes | yes | no |
| okd-1 | 192.168.149.130 | no | yes | yes | yes | no |
| okd-2 | 192.168.149.131 | no | yes | yes | no | yes |

前置步骤与快速安装相同，不同之处在于keepalived的安装配置以及inventory的配置

首先为三个主节点都安装keepalived

{% code title="okd-X:~/ \#" %}
```bash
yum install -y keepalived
```
{% endcode %}

开启ipv4转发

```bash
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
```

为keepalived添加防火墙策略

{% code title="okd-X:~/ \#" %}
```bash
firewall-cmd --direct --permanent --add-rule ipv4 filter INPUT 0 --in-interface ens33 --destination 224.0.0.18 --protocol vrrp -j ACCEPT
firewall-cmd --direct --permanent --add-rule ipv4 filter OUTPUT 0 --out-interface ens33 --destination 224.0.0.18 --protocol vrrp -j ACCEPT
firewall-cmd --reload
```
{% endcode %}

配置okd-0作为keepalived主节点，另外两个节点作为备用节点

{% tabs %}
{% tab title="okd-0:/etc/keepalived/keepalived.conf" %}
```text
global_defs {
	router_id LVS_okd-0
}

vrrp_instance VI_1 {
	state MASTER
	interface ens33
	virtual_router_id 1
	priority 100
	advert_int 1
	authentication {
		auth_type PASS
		auth_pass keepalivedpass
	}
	virtual_ipaddress {
		192.168.149.135/24
	}
}
```
{% endtab %}

{% tab title="okd-1:/etc/keepalived/keepalived.conf" %}
```
global_defs {
	router_id LVS_okd-1
}

vrrp_instance VI_1 {
	state BACKUP
	interface ens33
	virtual_router_id 1
	priority 99
	advert_int 1
	authentication {
		auth_type PASS
		auth_pass keepalivedpass
	}
	virtual_ipaddress {
		192.168.149.135/24
	}
}
```
{% endtab %}

{% tab title="okd-0:/etc/keepalived/keepalived.conf" %}
```
global_defs {
	router_id LVS_okd-2
}

vrrp_instance VI_1 {
	state BACKUP
	interface ens33
	virtual_router_id 1
	priority 99
	advert_int 1
	authentication {
		auth_type PASS
		auth_pass keepalivedpass
	}
	virtual_ipaddress {
		192.168.149.135/24
	}
}
```
{% endtab %}
{% endtabs %}

启动keepalived

{% code title="okd-X:~/ \#" %}
```bash
systemctl start keepalived
systemctl enable keepalived
```
{% endcode %}

这时okd-0节点上的ens33网络接口上会绑定一个新的地址，当okd-0节点故障时，ip地址会浮动到其他节点

配置inventory

```text
[OSEv3:children]
masters
nodes
etcd

[OSEv3:vars]
openshift_deployment_type=origin
openshift_release=3.9

openshift_master_cluster_method=native
openshift_master_cluster_hostname=192.168.149.135
openshift_master_cluster_public_hostname=192.168.149.135

osm_cluster_network_cidr=10.128.0.0/14
openshift_portal_net=172.30.0.0/16
osm_host_subnet_length=9
openshift_disable_check=disk_availability,memory_availability

# 配置多租户网络隔离
os_sdn_network_plugin_name='redhat/openshift-ovs-multitenant'
# Router服务的默认域名后缀
openshift_master_default_subdomain=apps.oc.local
# 配置docker日志的滚动清理策略和非加密镜像仓库地址
openshift_docker_options='--registry-mirror=https://53mhb806.mirror.aliyuncs.com --log-driver json-file --insecure-registry=172.30.0.0/16 --log-opt max-size=1M --log-opt max-file=3'

# 安装Hawkular,启用metrics
openshift_metrics_install_metrics=true
openshift_metrics_hawkular_hostname=hawkular-metrics.oc.local
openshift_metrics_image_version=v3.9

# 配置认证方式
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider', 'filename': '/etc/origin/master/htpasswd'}]

# 计算节点配置
openshift_node_kubelet_args={'pods-per-core': ['10'], 'max-pods': ['250'], 'image-gc-high-threshold': ['90'], 'image-gc-low-threshold': ['80']}

# 启用时钟同步
openshift_clock_enabled=true

[masters]
okd-0
okd-1
okd-2

[etcd]
okd-0
okd-1
okd-2

[nodes]
okd-0 openshift_schedulable=true openshift_node_labels="{'region': 'infra', 'zone': 'default'}"
okd-1 openshift_schedulable=true openshift_node_labels="{'region': 'infra', 'zone': 'default'}"
okd-2 openshift_schedulable=true openshift_node_labels="{'zone': 'default'}"
```

配置node域名解析

```bash
ansible nodes -m shell -a "mkdir -p /etc/origin/node"
ansible nodes -m shell -a "echo 'nameserver 8.8.8.8' > /etc/origin/node/resolv.conf"
```

安装预检查

{% code title="okd-0:/root/Downloads/openshift-ansible-openshift-ansible-3.9.99-1/ \#" %}
```bash
ansible-playbook playbooks/prerequisites.yml
```
{% endcode %}

安装集群

{% code title="okd-0:/root/Downloads/openshift-ansible-openshift-ansible-3.9.99-1/ \#" %}
```bash
ansible-playbook playbooks/deploy_cluster.yml
```
{% endcode %}

## 额外组件

### Prometheus

```text
ansible-playbook playbooks/openshift-prometheus/config.yml
```

### Grafana

```text
ansible-playbook playbooks/openshift-grafana/config.yml
```

### EFK

```text
ansible-playbook playbooks/openshift-logging/config.yml
```

