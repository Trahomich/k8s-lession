# ReplicaSet — поддержание количества подов

ReplicaSet гарантирует, что указанное количество replicas подов
всегда работает. Если pod упал — ReplicaSet создаст новый.

## Как работает

1. ReplicaSet использует **selector** для поиска подов по label
2. Сравнивает количество найденных подов с **replicas**
3. Если не хватает — создаёт недостающие
4. Если лишние — удаляет

## ReplicaSet vs Deployment

| | ReplicaSet | Deployment |
|--|-----------|------------|
| Масштабирование | Да | Да |
| Rolling update | **Нет** | Да |
| Rollback | **Нет** | Да |
| Использование | Редко напрямую | **Через Deployment** |

> В production ReplicaSet почти не используют напрямую.
> Deployment создаёт ReplicaSet под капотом.

## Манифест

```yaml
# replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
  namespace: lesson
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-rs
  template:
    metadata:
      labels:
        app: nginx-rs
    spec:
      containers:
      - name: nginx
        image: nginx:1.27
        ports:
        - containerPort: 80
```

## Важно: selector должен совпадать с labels шаблона

```yaml
# Правильно:
selector:
  matchLabels:
    app: nginx-rs    # ← совпадает
template:
  metadata:
    labels:
      app: nginx-rs  # ← совпадает

# Ошибка — ReplicaSet не запустится:
selector:
  matchLabels:
    app: nginx
template:
  metadata:
    labels:
      app: other     # ← не совпадает!
```

## Полезные команды

```bash
kubectl apply -f replicaset.yaml -n lesson

# Посмотреть ReplicaSet
kubectl get rs -n lesson

# Масштабировать
kubectl scale rs nginx-rs --replicas=5 -n lesson

# Удалить
kubectl delete rs nginx-rs -n lesson
```
