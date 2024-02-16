# Домашнее задание к занятию «Хранение в K8s. Часть 2»

### Цель задания

В тестовой среде Kubernetes нужно создать PV и продемострировать запись и хранение файлов.

------

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным GitHub-репозиторием.

------

### Дополнительные материалы для выполнения задания

1. [Инструкция по установке NFS в MicroK8S](https://microk8s.io/docs/nfs). 
2. [Описание Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/). 
3. [Описание динамического провижининга](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/). 
4. [Описание Multitool](https://github.com/wbitt/Network-MultiTool).

------

### Задание 1

**Что нужно сделать**

Создать Deployment приложения, использующего локальный PV, созданный вручную.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-app
  labels:
    app: deployment-app
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deployment-app
  template:
    metadata:
      labels:
        app: deployment-app
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ['sh', '-c', 'while true; do echo New-string >> /output/New-file.txt; sleep 5; done']
        volumeMounts:
          - name: vol
            mountPath: /output
        imagePullPolicy: IfNotPresent
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
          name: multitool
        volumeMounts:
        - name: vol
          mountPath: /input
        imagePullPolicy: IfNotPresent
      volumes:
      - name: vol
        persistentVolumeClaim:
          claimName: pvc-vol
```

2. Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.

| PV |
| :---: |
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-volume
  namespace: default
spec:
  storageClassName: manual
  volumeMode: Filesystem
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /mnt/disk/ssd1
  persistentVolumeReclaimPolicy: Retain
```

| PVC |
| :---: |
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-vol
  namespace: default
spec:
  storageClassName: manual
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

3. Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории. 

<p align="center">
    <img width="1200 height="600" src="/img/deploy-app-pv.png">
</p>

4. Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.

<p align="center">
    <img width="1200 height="600" src="/img/delete-deploy-pvc.png">
</p>

```
При отсутствии PVC статус PV изменился с Bound на Released.
```

5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV.  Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.
6. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

<p align="center">
    <img width="1200 height="600" src="/img/delete-pv.png">
</p>

```
При использовании локального хранилища в конфигурации PV, даже после удаления PV, файл остаётся в указанном каталоге.
```

------

### Задание 2

**Что нужно сделать**

Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.

1. Включить и настроить NFS-сервер на MicroK8S.

<p align="center">
    <img width="1200 height="600" src="/img/microk8s-enable-nfs.png">
</p>

2. Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-multitool
  labels:
    app: deployment-multitoolv
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deployment-multitool
  template:
    metadata:
      labels:
        app: deployment-multitool
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
          name: multitool
        volumeMounts:
        - name: vol
          mountPath: /input
        imagePullPolicy: IfNotPresent
      volumes:
      - name: vol
        persistentVolumeClaim:
          claimName: pvc-nfs
```

3. Продемонстрировать возможность чтения и записи файла изнутри пода. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

<p align="center">
    <img width="1200 height="600" src="/img/deploy-nfs-read-write.png">
</p>

------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
