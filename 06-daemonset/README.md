# DaemonSet — под на каждой ноде

DaemonSet гарантирует, что **копия pod запущена на каждой ноде** кластера
(или на подходящих по селектору нодах).

## Типичные применения

- Лог-коллекторы (Fluentd, Filebeat)
- Мониторинг-агенты (Prometheus Node Exporter, Datadog)
- Сетевые плагины (Calico, Flannel, CNI)
- Хранилище (csi-node, local-volume-provisioner)

## Как работает

1. При добавлении новой ноды — DaemonSet автоматически создаёт на ней pod
2. При удалении ноды — pod удаляется вместе с ней
3. По умолчанию — pod на КАЖДОЙ ноде (включая master)

## Манифест

```yaml
# daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
  namespace: lesson
spec:
  selector:
    matchLabels:
      app: log-collector
  template:
    metadata:
      labels:
        app: log-collector
    spec:
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        effect: NoSchedule    # запускать и на master-нодах
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.16
        resources:
          limits:
            memory: "200Mi"
            cpu: "100m"
          requests:
            memory: "100Mi"
            cpu: "50m"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: containers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: containers
        hostPath:
          path: /var/lib/docker/containers
```

## Ограничение по нодам (nodeSelector)

```yaml
# daemonset-nodeselector.yaml
spec:
  template:
    spec:
      nodeSelector:
        disktype: ssd    # запускать только на нодах с SSD
```

## Полезные команды

```bash
kubectl apply -f daemonset.yaml -n lesson

# Посмотреть DaemonSet
kubectl get ds -n lesson

# На каких нодах запущены поды
kubectl get pods -n lesson -l app=log-collector -o wide

# Обновление — по умолчанию RollingUpdate
kubectl set image daemonset/log-collector fluentd=fluent/fluentd:v1.17 -n lesson

# Статус обновления
kubectl rollout status ds/log-collector -n lesson
```
