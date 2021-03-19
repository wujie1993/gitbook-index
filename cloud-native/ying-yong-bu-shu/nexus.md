# Nexus

```text
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nexus-data
  namespace: repository
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 50Gi
  storageClassName: nfs-private
  volumeMode: Filesystem
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus
  namespace: repository
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nexus
  template:
    metadata:
      labels:
        app: nexus
    spec:
      securityContext:
        runAsUser: 200
        runAsGroup: 200
        fsGroup: 200
      containers:
      - image: sonatype/nexus3:3.30.0
        name: nexus
        ports:
        - containerPort: 8081
        volumeMounts:
        - name: data
          mountPath: /nexus-data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: nexus-data
      restartPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
  name: nexus
  namespace: repository
spec:
  ports:
  - name: "http"
    port: 8081
    targetPort: 8081
    nodePort: 30001
  selector:
    app: nexus
  type: NodePort

```

