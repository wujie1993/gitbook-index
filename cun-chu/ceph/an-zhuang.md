# 安装

## ceph-ansible

### 项目地址

{% embed url="https://github.com/ceph/ceph-ansible" %}

### 安装文档

{% embed url="https://docs.ceph.com/ceph-ansible/master/" %}

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
```
[mons]
centos7-dev

[osds]
centos7-dev
```
{% endcode %}

配置group\_vars

{% tabs %}
{% tab title="group_vars/all.yml" %}
```
---
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
ceph_repository: community
valid_ceph_repository:
  - community
ceph_mirror: http://download.ceph.com
ceph_stable_key: https://download.ceph.com/keys/release.asc
ceph_stable_release: nautilus
ceph_stable_repo: "{{ ceph_mirror }}/debian-{{ ceph_stable_release }}"
ceph_stable_redhat_distro: el7
generate_fsid: true
ceph_conf_key_directory: /etc/ceph
ceph_keyring_permissions: '0600'
cephx: true
monitor_interface: eth0
ip_version: ipv4
public_network: 172.17.209.112/28
cluster_network: "{{ public_network | regex_replace(' ', '') }}"
osd_mkfs_type: xfs
osd_mkfs_options_xfs: -f -i size=2048
osd_mount_options_xfs: noatime,largeio,inode64,swalloc
osd_objectstore: bluestore
dashboard_enabled: False-
```
{% endtab %}

{% tab title="group_vars/osds.yml" %}
```
---
devices:
  - /dev/sdb
  - /dev/sdc
  - /dev/sdd
```
{% endtab %}
{% endtabs %}

执行部署

```
cp site.yml.sample site.yml
ansible-playbook -i /etc/ansible/hosts-ceph-nautilus site.yml
```

检查集群状态`ceph -s`
