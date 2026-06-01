# Service — сетевой доступ к подам

Service (svc) — стабильная точка входа к набору подов.
Поды приходят и уходят, IP меняются — Service даёт постоянный адрес.

## Зачем нужен

- Поды имеют временные IP — при пересоздании IP меняется
- Service предоставляет **стабильный IP и DNS-имя**
- Распределяет трафик между подами (load balancing)

## Типы Service

| Тип | Описание |
|-----|----------|
| ClusterIP | Внутренний IP (default) — доступен только внутри кластера |
| NodePort | Доступен на порту каждой ноды (30000-32767) |
| LoadBalancer | Создаёт внешний балансировщик (cloud) |
| ExternalName | DNS CNAME-запись на внешний адрес |

## Как Service находит поды

Service использует **selector** для поиска подов по label.
Трафик направляется на `targetPort` контейнера.

```
Client → Service (ClusterIP:80) → pod-1 (targetPort:80)
                                → pod-2 (targetPort:80)
                                → pod-3 (targetPort:80)
```

## Манифест: ClusterIP (по умолчанию)

```yaml
# service-clusterip.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  namespace: lesson
spec:
  type: ClusterIP
  selector:
    app: nginx-deploy
  ports:
  - port: 80           # порт Service
    targetPort: 80     # порт контейнера
```

## Манифест: NodePort

```yaml
# service-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
  namespace: lesson
spec:
  type: NodePort
  selector:
    app: nginx-deploy
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080    # порт на ноде (30000-32767)
```

## Манифест: LoadBalancer

```yaml
# service-loadbalancer.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
  namespace: lesson
spec:
  type: LoadBalancer
  selector:
    app: nginx-deploy
  ports:
  - port: 80
    targetPort: 80
```

## DNS внутри кластера

```
<service>.<namespace>.svc.cluster.local

# Примеры:
nginx-svc.lesson.svc.cluster.local
nginx-svc.lesson               # сокращённо (в том же namespace)
nginx-svc                      # ещё короче (в том же namespace)
```

## Полезные команды

```bash
kubectl apply -f service-clusterip.yaml -n lesson

# Посмотреть сервисы
kubectl get svc -n lesson

# Проверить endpoints (поды за сервисом)
kubectl get endpoints nginx-svc -n lesson

# Проверить DNS изнутри кластера
kubectl run tmp --rm -it --image=busybox --restart=Never -- \
  wget -qO- http://nginx-svc.lesson:80
```
