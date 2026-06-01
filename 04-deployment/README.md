# Deployment — декларативное обновление

Deployment управляет ReplicaSet и обеспечивает:
- Rolling updates (бесшовное обновление)
- Rollback (откат к предыдущей версии)
- Масштабирование
- Self-healing (автовосстановление)

## Как устроен внутри

```
Deployment
  └── ReplicaSet (rev-2, текущая)
        ├── Pod-1
        ├── Pod-2
        └── Pod-3
  └── ReplicaSet (rev-1, предыдущая, scaled to 0)
```

При обновлении образа Deployment создаёт **новый ReplicaSet** и плавно
перекатывает поды со старого на новый.

## Стратегии обновления

| Стратегия | Описание |
|-----------|----------|
| RollingUpdate (default) | Постепенная замена: создаёт новые, удаляет старые |
| Recreate | Удаляет ВСЕ старые поды, затем создаёт новые (downtime!) |

## Манифест: Deployment с rolling update

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  namespace: lesson
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deploy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # сколько лишних подов можно создать
      maxUnavailable: 0   # сколько подов может быть недоступно
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
      - name: nginx
        image: nginx:1.27
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
```

## Манифест: Deployment с Recreate

```yaml
# deployment-recreate.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-recreate
  namespace: lesson
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-recreate
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nginx-recreate
    spec:
      containers:
      - name: nginx
        image: nginx:1.27
        ports:
        - containerPort: 80
```

## Типичный рабочий процесс

```bash
# Применить
kubectl apply -f deployment.yaml -n lesson

# Обновить образ (триггер rolling update)
kubectl set image deployment/nginx-deploy nginx=nginx:1.28 -n lesson

# Статус rollout
kubectl rollout status deployment/nginx-deploy -n lesson

# История релизов
kubectl rollout history deployment/nginx-deploy -n lesson

# Откат на предыдущую версию
kubectl rollout undo deployment/nginx-deploy -n lesson

# Откат на конкретную ревизию
kubectl rollout undo deployment/nginx-deploy --to-revision=2 -n lesson

# Масштабирование
kubectl scale deployment nginx-deploy --replicas=5 -n lesson
```
