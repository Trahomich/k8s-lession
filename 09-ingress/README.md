# Ingress — маршрутизация HTTP-трафика

Ingress — это L7-балансировщик (HTTP/HTTPS) внутри кластера.
Позволяет маршрутизировать трафик по hostname и path к разным сервисам.

## Зачем нужен

Без Ingress — для каждого сервиса нужен отдельный LoadBalancer (дорого).
С Ingress — один LoadBalancer, а маршрутизация — по правилам.

```
Internet → Ingress Controller (LoadBalancer)
              │
              ├─ host: app.example.com → Service A → Pods
              ├─ host: api.example.com → Service B → Pods
              └─ host: app.example.com/admin → Service C → Pods
```

## Ingress Controller

Ingress-ресурс — это только правила. Для работы нужен **Ingress Controller**:
- NGINX Ingress Controller (самый популярный)
- Traefik
- HAProxy
- Contour

## Манифест: Простой Ingress

```yaml
# ingress-simple.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: lesson
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
```

## Манифест: Мульти-хост с TLS

```yaml
# ingress-tls.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-ingress
  namespace: lesson
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.example.com
    - api.example.com
    secretName: tls-secret
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-svc
            port:
              number: 80
```

## Манифест: Path-based routing

```yaml
# ingress-paths.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-ingress
  namespace: lesson
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /web(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
      - path: /api(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: api-svc
            port:
              number: 80
```

## pathType

| Тип | Описание |
|-----|----------|
| Exact | Точное совпадение пути |
| Prefix | Совпадение по префиксу (разделяется по `/`) |
| ImplementationSpecific | Зависит от Ingress Controller |

## Полезные команды

```bash
kubectl apply -f ingress-simple.yaml -n lesson

# Посмотреть Ingress
kubectl get ingress -n lesson

# Описание (правила, IP, backend)
kubectl describe ingress nginx-ingress -n lesson

# Проверить (добавить в /etc/hosts)
# <EXTERNAL-IP> app.example.com
curl http://app.example.com
```
