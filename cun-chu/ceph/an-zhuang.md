# 安装

## ceph-ansible

### 项目地址

{% embed url="https://github.com/ceph/ceph-ansible" %}

### 快速安装

安装python-pip

```bash
yum install python-pip
```

下载脚本，以4.0版本为例（分支前缀stable）

```bash
git clone https://github.com/ceph/ceph-ansible.git
cd ceph-ansible
git checkout stable-4.0
```

安装python依赖包

```bash
pip install -r requirements.txt
```

配置inventory

{% code title="/etc/ansible/hosts-ceph-nautilus" %}
```text
[mons]
centos7-dev

[osds]
centos7-dev
```
{% endcode %}

配置group\_vars

{% code title="group\_vars/all.yml" %}
```text
ceph_release_num:
  nautilus: 14
fetch_directory: fetch/
cluster: ceph
mon_group_name: mons
osd_group_name: osds
configure_firewall: False
ceph_conf_local: true
centos_package_dependencies:
  - epel-release
  - libselinux-python
ntp_service_enabled: true
ntp_daemon_type: chronyd
bootstrap_dirs_owner: "64045"
bootstrap_dirs_group: "64045"
upgrade_ceph_packages: False
ceph_origin: repository
valid_ceph_origins:
  - repository
```
{% endcode %}

