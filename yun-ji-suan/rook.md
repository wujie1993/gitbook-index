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
---
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

为ceph-dashboard服务添加ingress路由

```text
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: rook-ceph-mgr-dashboard
  namespace: rook-ceph
  annotations:
    kubernetes.io/ingress.class: "nginx"
    kubernetes.io/tls-acme: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
    nginx.ingress.kubernetes.io/server-snippet: |
      proxy_ssl_verify off;
spec:
  tls:
   - hosts:
     - rook-ceph.minikube.local
     secretName: rook-ceph.minikube.local
  rules:
  - host: rook-ceph.minikube.local
    http:
      paths:
      - path: /
        backend:
          serviceName: rook-ceph-mgr-dashboard
          servicePort: https-dashboard
```

将域名rook-ceph.minikube.local加入/etc/hosts后通过浏览器访问

{% embed url="https://rook-ceph.minikube.local/" %}

### 使用rbd存储

创建rbd存储池

```text
---
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

创建完成后在rook-ceph-tools中使用指令`ceph osd pool ls`可以看到新建了以下存储池

* replicapool

创建以rbd为存储的storageclass

```text
---
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

使用statefulset测试通过storageclass挂载rbd存储

```text
---
kind: StatefulSet
apiVersion: apps/v1
metadata:
  name: storageclass-rbd-test
  namespace: default
  labels:
    app: storageclass-rbd-test
spec:
  replicas: 2
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
---
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

创建完成后在rook-ceph-tools中使用指令`ceph osd pool ls`可以看到新建了以下存储池

* myfs-metadata
* myfs-data0

创建以cephfs为存储的storageclass

```text
---
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

使用deployment测试通过storageclass挂载cephfs共享存储

```text
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: data-storageclass-cephfs-test
  namespace: default
  labels:
    app: storageclass-cephfs-test
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: csi-cephfs
  volumeMode: Filesystem
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: storageclass-cephfs-test
  namespace: default
  labels:
    app: storageclass-cephfs-test
spec:
  replicas: 2
  selector:
    matchLabels:
      app: storageclass-cephfs-test
  template:
    metadata:
      labels:
        app: storageclass-cephfs-test
    spec:
      restartPolicy: Always
      containers:
        - name: storageclass-cephfs-test
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - name: data
              mountPath: /data
          image: 'centos:7'
          args:
            - 'sh'
            - '-c'
            - 'sleep 3600'
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: data-storageclass-cephfs-test
```

### 使用s3存储

创建对象存储网关与存储池

```text
---
apiVersion: ceph.rook.io/v1
kind: CephObjectStore
metadata:
  name: my-store
  namespace: rook-ceph
spec:
  metadataPool:
    failureDomain: osd
    replicated:
      size: 3
  dataPool:
    failureDomain: osd
    replicated:
      size: 3
  preservePoolsOnDelete: false
  gateway:
    type: s3
    sslCertificateRef:
    port: 80
    securePort:
    instances: 1
    placement:
    annotations:
    resources:
```

创建完成后在rook-ceph-tools中使用指令`ceph osd pool ls`可以看到新建了以下存储池

* .rgw.root
* my-store.rgw.buckets.data
* my-store.rgw.buckets.index
* my-store.rgw.buckets.non-ec
* my-store.rgw.control
* my-store.rgw.log
* my-store.rgw.meta

为ceph-rgw服务添加ingress路由

```text
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: rook-ceph-rgw
  namespace: rook-ceph
  annotations:
    kubernetes.io/ingress.class: "nginx"
    kubernetes.io/tls-acme: "true"
spec:
  tls:
   - hosts:
     - rook-ceph-rgw.minikube.local
     secretName: rook-ceph-rgw.minikube.local
  rules:
  - host: rook-ceph-rgw.minikube.local
    http:
      paths:
      - path: /
        backend:
          serviceName: rook-ceph-rgw-my-store
          servicePort: http
```

将域名rook-ceph-rgw.minikube.local加入/etc/hosts后通过浏览器访问

{% embed url="https://rook-ceph-rgw.minikube.local/" %}

添加对象存储用户

```text
---
apiVersion: ceph.rook.io/v1
kind: CephObjectStoreUser
metadata:
  name: my-user
  namespace: rook-ceph
spec:
  store: my-store
  displayName: "my display name"
```

创建对象存储用户的同时会生成AccessKey和SecretKey，以secret的方式保存

获取AccessKey

```text
kubectl get secret rook-ceph-object-user-my-store-my-user -n rook-ceph -o jsonpath='{.data.AccessKey}'|base64 -d
```

获取SecretKey

```text
kubectl get secret rook-ceph-object-user-my-store-my-user -n rook-ceph -o jsonpath='{.data.SecretKey}'|base64 -d
```

根据上述步骤获取到的信息使用S3客户端进行连接

![](../.gitbook/assets/image%20%282%29.png)

创建以s3为存储的storageclass

```text
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: rook-ceph-delete-bucket
provisioner: ceph.rook.io/bucket
reclaimPolicy: Delete
parameters:
  objectStoreName: my-store
  objectStoreNamespace: rook-ceph
  region: default
```

为storageclass创建对应的存储桶

```text
apiVersion: objectbucket.io/v1alpha1
kind: ObjectBucketClaim
metadata:
  name: ceph-delete-bucket
spec:
  generateBucketName: ceph-bkt
  storageClassName: rook-ceph-delete-bucket
```

