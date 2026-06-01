# 10. Cert-Manager и TLS-сертификаты для Ingress

## Что такое cert-manager

[cert-manager](https://cert-manager.io/) — контроллер для Kubernetes, который автоматически выпускает и обновляет TLS-сертификаты. Работает с различными Issuer-ами, самый популярный — **Let's Encrypt** (бесплатные сертификаты).

## Компоненты

| Ресурс | Назначение |
|---|---|
| **Issuer** | Выпускает сертификаты в одном namespace |
| **ClusterIssuer** | Выпускает сертификаты во всём кластере |
| **Certificate** | Запрос на сертификат (домен, секрет, issuer) |
| **CertificateRequest** | Внутренний ресурс (создаётся автоматически) |
| **Order / Challenge** | Процесс прохождения ACME-проверки |

## Типы ACME-проверок

| Тип | Описание |
|---|---|
| **HTTP-01** | Let's Encrypt обращается к `http://<домен>/.well-known/acme-challenge/...` — требует публичный Ingress |
| **DNS-01** | cert-manager создаёт TXT-запись в DNS — работает без публичного доступа, поддерживает wildcard |

## Пошаговая установка

### 1. Установить cert-manager

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.16.3/cert-manager.yaml
```

Проверка:

```bash
kubectl get pods -n cert-manager
```

Все поды должны быть Running.

### 2. Создать ClusterIssuer

Для **тестирования** (сертификаты не доверенные, но лимиты не бьются):

```bash
kubectl apply -f cluster-issuer-staging.yaml
```

Для **продакшена**:

```bash
kubectl apply -f cluster-issuer-prod.yaml
```

### 3. Annotировать Ingress

cert-manager ищет аннотацию на Ingress и автоматически создаёт Certificate:

```yaml
annotations:
  cert-manager.io/cluster-issuer: letsencrypt-prod
```

См. `ingress-with-tls.yaml` — полностью готовый манифест.

### 4. Проверить сертификат

```bash
# Статус Certificate
kubectl get certificate -n lesson

# Подробности
kubectl describe certificate my-app-cert -n lesson

# Логи cert-manager
kubectl logs -n cert-manager -l app=cert-manager --tail=50
```

## Важные моменты

- Для HTTP-01 challenge Ingress должен быть доступен из интернета по домену
- Сертификаты Let's Encrypt действительны 90 дней, cert-manager обновляет автоматически
- Начинай со staging-issuer, иначе быстро упрёшься в rate limit Let's Encrypt
- DNS-01 требует настроенного DNS01 solver (Cloudflare, Route53 и т.д.)
- Секрет с сертификатом создаётся в том же namespace, что и Ingress
