В Kubernetes MCS возможно использование как block storage, так и file storage.

Файловое хранилище может быть реализовано на базе контейнера, содержащего NFS-сервер и используещего persistent volume для хранения данных.
При этом, подключение данного файлового хранилища может быть реализовано с помощью отдельного StorageClass, динамически создающего новый Volume на NFS-сервере.

1. Используйте helm, чтобы установить [nfs-provisioner](https://hub.kubeapps.com/charts/stable/nfs-server-provisioner) . В параметре persistence.size необходимо указать размер создаваемого диска, в параметре persistence.storageClass класс хранения. Допустимы значения ssd для быстрого доступа и hdd для обыкновенного доступа.
```
helm install stable/nfs-server-provisioner --version 0.1.4 --name nfs-provisioner --set persistence.enabled=true,persistence.size=200Gi,persistence.storageClass="ssd",storageClass.name="nfs"
```
После установки будут созданы:
 1) Новый StorageClass с именем "nfs"
 2) NFS-сервер со следующими открытыми портами:
 - 2049, на котором слушает сервис NFS
 - 20048, на котором слушает сервис mountd
 - 51413, на котором слушает сервис RPC


2. Создайте PersistentVolumeClaim следующего вида:

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nfs-pvc
  namespace: default
  annotations:
    storageClassName: nfs
    volume.beta.kubernetes.io/storage-class: nfs
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
```

В поле storage укажите требуемый размер диска. Также обратите внимание на то, что данный PVC будет доступен только в namespace, в котором вы его создали.

```
kubectl apply -f nfs-claim.yaml 
```

3. Подключите созданный PVC к pod следующим образом:

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-test-nfs
spec:
  replicas: 1
  selector:
    matchLabels:
      io.kompose.service: nginx
      app: nginx
  template:
    metadata:
      labels:
        io.kompose.service: nginx
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        volumeMounts:
        - name: data
          mountPath: /opt/data
      restartPolicy: Always
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: nfs-pvc
```