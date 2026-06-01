# Pod — минимальная единица K8s

Pod — группа из одного или нескольких контейнеров с общим сетевым стеком и хранилищем.
Это атомарная единица развёртывания в Kubernetes.

## Ключевые концепции

- **Один IP-адрес** на pod — все контейнеры внутри pod делят сеть
- **Общие volumes** — контейнеры в одном pod могут делить файлы
- **Ephemeral** — pod не переживёт гибель ноды (используйте Deployment)
- **Не создавайте pod напрямую в production** — используйте Deployment/ReplicaSet

## Жизненный цикл Pod

```
Pending → Running → Succeeded / Failed
                 ↘ CrashLoopBackOff (ошибка)
```

| Фаза | Описание |
|------|----------|
| Pending | Создан, но ещё не запущен (ждёт образ/ноду) |
| Running | Привязан к ноде, контейнеры работают |
| Succeeded | Все контейнеры завершились с кодом 0 |
| Failed | Хотя бы один контейнер завершился с ошибкой |
| Unknown | Не удалось получить состояние |

## Манифест: Простой pod

```yaml
# pod-nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: lesson
  labels:
    app: nginx
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

## Манифест: Multi-container pod

```yaml
# pod-multi.yaml — паттерн sidecar
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
  namespace: lesson
  labels:
    app: demo
spec:
  containers:
  - name: app
    image: nginx:1.27
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  - name: log-collector
    image: busybox:1.36
    command: ["sh", "-c", "tail -f /var/log/nginx/access.log"]
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  volumes:
  - name: shared-logs
    emptyDir: {}
```

## Манифест: Pod с probes

```yaml
# pod-probes.yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-health
  namespace: lesson
  labels:
    app: health-demo
spec:
  containers:
  - name: nginx
    image: nginx:1.27
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 5
    startupProbe:
      httpGet:
        path: /
        port: 80
      failureThreshold: 30
      periodSeconds: 2
```

## Полезные команды

```bash
# Создать pod
kubectl apply -f pod-nginx.yaml -n lesson

# Посмотреть поды
kubectl get pods -n lesson -o wide

# Логи
kubectl logs nginx -n lesson

# Логи конкретного контейнера (multi-container)
kubectl logs app-with-sidecar -c log-collector -n lesson

# Пробросить порт
kubectl port-forward nginx 8080:80 -n lesson

# Зайти в контейнер
kubectl exec -it nginx -- /bin/bash -n lesson

# Описание pod (события, IP, нода)
kubectl describe pod nginx -n lesson

# Удалить pod
kubectl delete pod nginx -n lesson
```
