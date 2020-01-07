# Rook

## 官方网站

{% embed url="https://rook.io/" %}

## 项目地址

{% embed url="https://github.com/rook/rook" %}

## 快速上手

### 安装集群

| 硬盘符号 | 大小 | 作用 |
| :--- | :--- | :--- |
| sdb | 50GB | OSD Data |
| sdc | 50GB | OSD Data |
| sdd | 50GB | OSD Data |
| sde | 50GB | OSD Metadata |

{% hint style="info" %}
安装前使用命令`lvm lvs`,`lvm vgs`和`lvm pvs`检查上述硬盘是否已经被使用，若已经使用需要删除，且确保硬盘上不存在分区和文件系统
{% endhint %}

前置准备

```text
modprobe rbd
yum install -y lvm2
```

安装operator

```text
git clone --single-branch --branch release-1.2 https://github.com/rook/rook.git
cd rook/cluster/examples/kubernetes/ceph
kubectl create -f common.yaml
kubectl create -f operator.yaml
```

安装ceph集群

```text
apiVersion: ceph.rook.io/v1
kind: CephCluster
metadata:
  name: rook-ceph
  namespace: rook-ceph
spec:
  cephVersion:
    image: ceph/ceph:v14.2.5
    allowUnsupported: false
  dataDirHostPath: /var/lib/rook
  skipUpgradeChecks: false
  mon:
    count: 3
    allowMultiplePerNode: true
  mgr:
    modules:
    - name: pg_autoscaler
      enabled: true
  dashboard:
    enabled: true
    ssl: true
  monitoring:
    enabled: false
    rulesNamespace: rook-ceph
  network:
    hostNetwork: false
  rbdMirroring:
    workers: 0
  annotations:
  resources:
  removeOSDsIfOutAndSafeToRemove: false
    useAllNodes: false
    useAllDevices: false
    config:
    nodes:
    - name: "minikube"
      devices:
      - name: "sdb"
      - name: "sdc"
      - name: "sdd"
      config:
        storeType: bluestore
        metadataDevice: "sde"
        databaseSizeMB: "1024"
        journalSizeMB: "1024"
        osdsPerDevice: "1"
  disruptionManagement:
    managePodBudgets: false
    osdMaintenanceTimeout: 30
    manageMachineDisruptionBudgets: false
    machineDisruptionBudgetNamespace: openshift-machine-api
```

安装命令行工具

```text
kubectl create -f toolbox.yaml
```

在toolbox中使用命令`ceph -s`查看集群状态

{% hint style="info" %}
在重装ceph集群时需要清理rook数据目录（默认：/var/lib/rook）
{% endhint %}

### 使用rbd存储

创建rbd存储池

```text
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  failureDomain: osd
  replicated:
    size: 3
```

{% hint style="info" %}
由于仅有一个节点和三个OSD，因此采用osd作为故障域
{% endhint %}

创建以rbd为存储的storageclass

```text
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: rook-ceph-block
provisioner: rook-ceph.rbd.csi.ceph.com
parameters:
  clusterID: rook-ceph
  pool: replicapool
  imageFormat: "2"
  imageFeatures: layering
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-rbd-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph
  csi.storage.k8s.io/fstype: ext4
reclaimPolicy: Delete
```

测试通过storageclass挂载rbd存储

```text
kind: StatefulSet
apiVersion: apps/v1
metadata:
  name: storageclass-rbd-test
  namespace: default
  labels:
    app: storageclass-rbd-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: storageclass-rbd-test
  template:
    metadata:
      labels:
        app: storageclass-rbd-test
    spec:
      restartPolicy: Always
      containers:
        - name: storageclass-rbd-test
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - name: data
              mountPath: /data
          image: 'centos:7'
          args:
            - 'sh'
            - '-c'
            - 'sleep 3600'
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
        storageClassName: rook-ceph-block
```

### 使用cephfs存储

创建mds服务与cephfs文件系统myfs

```text
apiVersion: ceph.rook.io/v1
kind: CephFilesystem
metadata:
  name: myfs
  namespace: rook-ceph
spec:
  metadataPool:
    failureDomain: osd
    replicated:
      size: 3
  dataPools:
    - failureDomain: osd
      replicated:
        size: 3
  preservePoolsOnDelete: true
  metadataServer:
    activeCount: 1
    activeStandby: true
    placement:
    annotations:
    resources:
```

> 创建完成后在rook-ceph-tools中使用指令`ceph osd pool ls`可以看到新建了两个存储池myfs-metadata和myfs-data0

创建以cephfs为存储的storageclass

```text
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: csi-cephfs
provisioner: rook-ceph.cephfs.csi.ceph.com
parameters:
  clusterID: rook-ceph
  fsName: myfs
  pool: myfs-data0
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph
reclaimPolicy: Delete
mountOptions:
```



