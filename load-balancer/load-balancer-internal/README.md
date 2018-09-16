Этот пример показывает как создать сервис, доступный с помощью `LoadBalancer`, но без внешнего IP-адреса.
Аннотация `service.beta.kubernetes.io/openstack-internal-load-balancer: "true"` активирует данное поведение: вместо публичного IP-адреса, будет выделенен внутренний. 
Это может быть полезно в гибридных сценариях, когда потребителями сервиса являются приложения во внутренней сети за рамками кластера Kubernetes.


```bash
$ kubectl create -f load-balancer-internal/service-internal-lb.yaml
```


После установки манифеста
```bash
$ watch kubectl get service
NAME                 CLUSTER-IP     EXTERNAL-IP       PORT(S)        AGE
nginx-internal-lb   10.0.0.10      192.168.0.181     80:30000/TCP   5m
``` 