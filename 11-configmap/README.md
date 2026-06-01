# ConfigMap

Конфигурация приложения без пересборки образа. ConfigMap хранит **нечувствительные** данные — настройки, конфиги, переменные окружения.

## Зачем

- Вынести конфигурацию из образа — один образ для dev/staging/prod
- Менять настройки без пересборки (перезапуск pod = новый конфиг)
- Шарить конфигурацию между подами

## 3 способа передать ConfigMap в pod

### 1. Переменные окружения (env / envFrom)

```yaml
env:
  - name: APP_ENV
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_ENV
envFrom:                    # все ключи сразу
  - configMapRef:
      name: app-config
```

### 2. Файл через volume

```yaml
volumeMounts:
  - name: config-volume
    mountPath: /etc/nginx/conf.d/default.conf
    subPath: nginx.conf      # один файл, не директорию
volumes:
  - name: config-volume
    configMap:
      name: nginx-config
```

### 3. Аргументы команды

```yaml
command: ["nginx", "-g", "daemon off;"]
# подставить через $(VAR) в args
```

## Применение

```bash
kubectl apply -f configmap-literal.yaml
kubectl apply -f configmap-file.yaml
kubectl apply -f pod-env-configmap.yaml
```

## Проверка

```bash
# посмотреть содержимое
kubectl get configmap app-config -n lesson -o yaml

# проверить переменные внутри pod
kubectl logs app-with-env -n lesson | grep APP_

# конфиг nginx
kubectl exec nginx-with-config -n lesson -- cat /etc/nginx/conf.d/default.conf
```

## Импорт из файла

```bash
# создать ConfigMap из файла на диске
kubectl create configmap my-config \
  --from-file=my-app.conf \
  --from-literal=DEBUG=true \
  -n lesson --dry-run=client -o yaml > my-config.yaml
```

## Подводные камни

- ConfigMap монтируется как том — обновление **не мгновенное** (до 60 сек синхронизация kubelet)
- Обновление ConfigMap **не перезапускает** pod автоматически — нужен restart Deployment
- Размер лимит — 1 MiB на ConfigMap
- НИКОГДА не хранить пароли/токены — для этого есть Secret
