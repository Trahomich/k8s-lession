# Namespace — изоляция ресурсов

Namespace (пространство имён) — логическая分组ировка ресурсов внутри кластера.
Позволяет нескольким командам использовать один кластер без конфликтов.

## Зачем нужен

- Разделение сред (dev, staging, prod)
- Квоты ресурсов на команду/проект
- Ограничения RBAC по namespace
- Избежание конфликтов имён

## Стандартные namespace

| Namespace | Назначение |
|-----------|-----------|
| `default` | Всё что не указало namespace |
| `kube-system` | Компоненты самого K8s |
| `kube-public` | Публичные данные (configmaps) |
| `kube-node-lease` | Heartbeat-данные нод |

## Манифест

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: lesson
```

## Применение

```bash
kubectl apply -f namespace.yaml

# Посмотреть все namespace
kubectl get namespaces

# Работать с ресурсами в namespace
kubectl get all -n lesson
```

## Полезные команды

```bash
# Текущий контекст с namespace
kubectl config set-context --current --namespace=lesson

# Все ресурсы в namespace
kubectl get all -n lesson

# Квота на namespace
kubectl describe resourcequota -n lesson
```
