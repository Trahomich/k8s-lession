# StatefulSet — stateful-приложения

StatefulSet предназначен для приложений, которым важна **идентичность** и **стабильность**:
- базы данных (MySQL, PostgreSQL, MongoDB)
- брокеры сообщений (Kafka, RabbitMQ)
- распределённые хранилища (Redis Cluster, Cassandra)

## Отличия от Deployment

| | Deployment | StatefulSet |
|--|-----------|-------------|
| Имена подов | случайные (nginx-deploy-7b8f...) | предсказуемые (web-0, web-1, web-2) |
| Порядок запуска | параллельный | последовательный (0 → 1 → 2) |
| Порядок удаления | случайный | обратный (2 → 1 → 0) |
| PVC | общий (если есть) | отдельный на каждый pod |
| Идентичность | нет | стабильный hostname + ordinal index |

## Гарантии StatefulSet

1. **Стабильное имя**: `web-0`, `web-1`, `web-2`
2. **Стабильный hostname**: внутри pod `hostname` = имя pod
3. **Стабильный PVC**: при пересоздании pod получит тот же диск
4. **Упорядоченный запуск/остановка**: pod-N не запустится пока pod-(N-1) не готов

## Манифест

```yaml
# statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
  namespace: lesson
spec:
  serviceName: web-headless    # обязательно! headless service
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.27
        ports:
        - containerPort: 80
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:        # отдельный PVC на каждый pod
  - metadata:
      name: www
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

## Headless Service (обязателен)

```yaml
# headless-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-headless
  namespace: lesson
spec:
  clusterIP: None              # это делает service headless
  selector:
    app: web
  ports:
  - port: 80
```

С headless service DNS-запись `web-0.web-headless.lesson.svc.cluster.local`
указывает напрямую на IP pod.

## Как это выглядит

```
web-0  →  PVC www-web-0  (1Gi)
web-1  →  PVC www-web-1  (1Gi)
web-2  →  PVC www-web-2  (1Gi)
```

## Полезные команды

```bash
kubectl apply -f headless-service.yaml -n lesson
kubectl apply -f statefulset.yaml -n lesson

# Наблюдать за последовательным запуском
kubectl get pods -w -n lesson -l app=web

# Посмотреть PVC
kubectl get pvc -n lesson

# Посмотреть StatefulSet
kubectl get sts -n lesson
```
