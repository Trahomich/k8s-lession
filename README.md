# Kubernetes: Начало

Практический курс по основным объектам Kubernetes с примерами манифестов.

## Структура

```
.
├── 01-namespace/
├── 02-pod/
├── 03-replicaset/
├── 04-deployment/
├── 05-statefulset/
├── 06-daemonset/
├── 07-pvc/
├── 08-service/
├── 09-ingress/
└── README.md
```

## Порядок изучения

1. **Namespace** — изоляция ресурсов внутри кластера
2. **Pod** — минимальная единица развертывания
3. **ReplicaSet** — поддержание нужного количества подов
4. **Deployment** — декларативное обновление подов
5. **StatefulSet** — stateful-приложения с устойчивой идентичностью
6. **DaemonSet** — под на каждой ноде
7. **PVC** — постоянное хранилище
8. **Service** — сетевой доступ к подам
9. **Ingress** — маршрутизация HTTP-трафика

## Быстрый старт

```bash
# Применить манифест
kubectl apply -f <файл.yaml>

# Посмотреть ресурсы
kubectl get all -n lesson

# Удалить всё
kubectl delete -f <файл.yaml>
```
