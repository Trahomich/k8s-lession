# Secrets

Secret — аналог ConfigMap, но для **чувствительных** данных: пароли, токены, ключи, сертификаты.

## В чём отличие от ConfigMap

- Значения хранятся в base64 (это **не шифрование**, только кодировка)
- etcd может шифровать Secrets на диске (нужна конфигурация EncryptionConfiguration)
- RBAC — можно ограничить кто читает Secrets
- Логи и describe подставляют `***` вместо значений

## Типы Secret

| Тип | Назначение |
|-----|-----------|
| `Opaque` | Произвольные данные (по умолчанию) |
| `kubernetes.io/dockerconfigjson` | Доступ к приватному registry |
| `kubernetes.io/tls` | TLS сертификат + ключ |
| `kubernetes.io/basic-auth` | Логин/пароль |
| `kubernetes.io/ssh-auth` | SSH ключи |
| `kubernetes.io/docker-registry` | Устаревший, используй dockerconfigjson |

## 2 формата данных

```yaml
data:           # base64 — вы кодируете сами
  DB_PASS: bXlwYXNz

stringData:     # plaintext — K8s закодирует сам
  DB_PASS: mypass
```

## Создание

```bash
# из literal
kubectl create secret generic db-secret \
  --from-literal=DB_PASSWORD=mypass \
  --from-literal=DB_USER=app_user \
  -n lesson --dry-run=client -o yaml > secret.yaml

# из файла
kubectl create secret generic tls-secret \
  --from-file=tls.crt=server.crt \
  --from-file=tls.key=server.key \
  -n lesson

# для docker registry
kubectl create secret docker-registry registry-secret \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass \
  -n lesson
```

## Использование в pod

### Переменные окружения

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: DB_PASSWORD
```

### Volume (как файлы)

```yaml
volumeMounts:
  - name: secret-volume
    mountPath: /etc/secrets
    readOnly: true
volumes:
  - name: secret-volume
    secret:
      secretName: db-secret
```

## Применение

```bash
kubectl apply -f secret-opaque.yaml
kubectl apply -f pod-with-secret.yaml
kubectl apply -f pod-secret-volume.yaml
```

## Проверка

```bash
# список secrets
kubectl get secrets -n lesson

# содержимое (значения скрыты)
kubectl describe secret db-secret -n lesson

# декодировать значение
kubectl get secret db-secret -n lesson \
  -o jsonpath='{.data.DB_PASSWORD}' | base64 -d

# проверить в pod
kubectl logs app-with-secret -n lesson
```

## Безопасность

- **base64 ≠ шифрование** — `base64 -d` раскодирует любой secret
- Включите шифрование etcd: `EncryptionConfiguration` с AES-CBC или AES-GCM
- Ограничьте RBAC: `kubectl auth can-i get secrets -n lesson`
- Используйте External Secrets Operator для HashiCorp Vault / AWS Secrets Manager
- Смотрите на Sealed Secrets (Bitnami) для шифрования прямо в git
- Ограничьте `automountServiceAccountToken: false` если pod не обращается к API

## Подводные камни

- Размер лимит — 1 MiB на Secret
- Обновление Secret в volume — **не мгновенное**, синхронизация kubelet
- Как и ConfigMap — обновление Secret не перезапускает pod
- `stringData` — write-only, при `kubectl get -o yaml` увидите только `data` (base64)
