# Persistent volumes и StatefulSet
StatefulSet предоставляют очень удобный способ работы со Stateful-приложениями, которым требуется обрабатывать события об остановке работы Pod и осуществления Graceful Shutdown. Достаточно часто подобными приложениями являются базы данных и очереди сообщений, которые работают в нескольких экземплярах, синхронизируемых друг с другом посредством репликации или кластеризации.

Существует несколько способов организации таких схем: с использованием общих Persistent Volumes (RWX) и с помощью индивидуальных Persistent Volumes (RWO).

В случае RWX Persistent Volumes (NFS, GlusterFS) вы можете использовать существующий Persistent Volume Claim в декларации StatefulSet.
Однако, такой способ не подойдет для RWO Persistent Volumes (Block Storage), т.к. блочное хранилище монтируется в эксклюзивном режиме к каждому Pod. В результате, если StatefulSet будет иметь 2 или более реплики Persistent Volume будет примонтирован только к 1 из Pod, а монтирование ко всем остальным Pod завершится неудачей.  

Для того, чтобы избежать подобных проблем, вы должны использовать `volumeClaimTemplates` в декларации StatefulSet.

```yaml
  volumeClaimTemplates:
  - metadata:
      name: data
      annotations:
        volume.alpha.kubernetes.io/storage-class: "ssd"
      labels:
        app: app-selector
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 8Gi
```


В этом случае, для каждой реплики StatefulSet будет создан PVC с именем data-0, data-1, data-2 итд в зависимости от количества реплик.
Каждый из этих PVC выделит Persistent Volume нужного объема, который и будет примонтирован к подам, соответствующим репликам StatefulSet.


