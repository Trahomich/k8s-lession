# PersistentVolumeClaim — постоянное хранилище

PVC — запрос на хранилище от пользователя. K8s автоматически находит
подходящий PersistentVolume (PV) или создаёт его через StorageClass.

## Зачем нужно

По умолчанию данные в контейнере теряются при перезапуске pod.
PVC даёт постоянный диск, который живёт независимо от pod.

## Иерархия

```
Pod → PVC → PV → Реальное хранилище (диск, NFS, cloud volume...)
         ↑
    StorageClass (динамическое выделение)
```

## AccessModes

| Режим | Аббревиатура | Описание |
|-------|-------------|----------|
| ReadWriteOnce | RWO | Одна нода — чтение/запись |
| ReadOnlyMany | ROX | Многие ноды — только чтение |
| ReadWriteMany | RWX | Многие ноды — чтение/запись |
| ReadWriteOncePod | RWOP | Один pod — чтение/запись (K8s 1.27+) |

## Манифест: PVC

```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
  namespace: lesson
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: standard   # зависит от кластера
  resources:
    requests:
      storage: 5Gi
```

## Манифест: Pod с PVC

```yaml
# pod-with-pvc.yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-storage
  namespace: lesson
spec:
  containers:
  - name: nginx
    image: nginx:1.27
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: data-pvc
```

## Полезные команды

```bash
# Посмотреть PVC
kubectl get pvc -n lesson

# Посмотреть PV
kubectl get pv

# Посмотреть StorageClass
kubectl get sc

# Описание PVC
kubectl describe pvc data-pvc -n lesson

# Удалить PVC (освободит диск!)
kubectl delete pvc data-pvc -n lesson
```
