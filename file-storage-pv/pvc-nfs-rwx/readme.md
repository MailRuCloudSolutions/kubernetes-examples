Вместе с блочной системой хранения на базе Ceph вы можете использовать общее файловое хранилище, доступное по протоколу NFS, для создания Persistent volumes в вашем кластере Kubernetes.

Для этого сначала необходимо создать NFS-диск в приватной подсети, ассоциированной с вашим кластером Kubernetes:

1. Выберите пункт меню «Вычислительные ресурсы» → «Общие ресурсы».
2. Перейдите на вкладку «Сети общих ресурсов».
3. Создайте новую Сеть общего ресурса, указав:
- Сеть Neutron =	Сеть кластера Kubernetes
- Подсеть Neutron =	Подсеть кластера Kubernetes

4. Перейдите на вкладку «Общие ресурсы».
5. Создайте новый «Общий ресурс», указав:
- Протокол общего ресурса =	NFS
- Размер =	Необходимый размер
- Сеть общего доступа =	Созданную ранее сеть
6. В меню действий созданного ресурса выберите пункт "Управление правилами"
- Нажмите "Добавить правило"
- В поле "Тип доступа" выберите значение "ip"
- В поле "Доступ к" введите значение подсети, выделенной для вашего кластера Kubernetes. Например, "10.1.1.0/24"

7. Перейдите в созданный ресурс и скопируйте значение поля «Путь». Например, 10.0.0.16:/shares/share-51831cd4-2fe9-4f08-ac45-6be754556519.
Далее вам нужно создать Persistent Volume используя манифест следующего вида (в поле «Server» нужно вписать IP-адрес из значения «Путь» созданного ранее ресурса).

Также вам нужно создать PersistentVolumeClaim, который использует ранее созданный NFS Persistent Volume в качестве источника.
Важно, чтобы вы указали в поле mountOptions `nfsvers=4.0` Persistent Volume, а также `storageClassName: ""` в декларации Persistent Volume Claim.

Ниже пример манифеста, создающего Persistent Volume на базе NFS и Persistent Volume Claim, использующий ранее созданный Persistent Volume в качестве источника.


```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: portal-info-data-pv
spec:
  accessModes:
    - ReadWriteMany
  mountOptions:
    - hard
    - nfsvers=4.0
    - timeo=60
    - retrans=10
  capacity:
    storage: 100Gi
  nfs:
    server: 10.0.0.28
    path: "/shares/share-76637908-0da9-4007-be78-e334ca4573a1"
  persistentVolumeReclaimPolicy: "Recycle"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: portal-info-data-pvc
spec:
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  volumeName: "portal-info-data-pv"
  storageClassName: ""

---
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 4
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: portal-info-data
          mountPath: /var/www/html
      restartPolicy: Always
      volumes:
      - name: portal-info-data
        persistentVolumeClaim:
          claimName: portal-info-data-pvc

```