# RabbitMQ

通过 Operator 方式部署

1、部署 operator

```text
kubectl apply -f https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml
```

2、部署 rabbitmq

```text
apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: custom-configuration
spec:
  replicas: 1
  rabbitmq:
    additionalConfig: |
      log.console.level = debug
```



