Этот пример показывает как создать сервис, доступный с помощью `LoadBalancer`, который перенаправляет трафик на целевые поды не с помощью Round-Robin балансировки, а с учетом предыдущих запросов одного и того же клиента. Это может помочь для решения многих проблем с традиционными stafetul веб-приложениями.
Параметр `sessionAffinity: ClientIP` активирует так называемый Session Affinity, т.е. все запросы одного и того же пользователя будут идти на один и тот же под до тех пор пока этот под жив.


```bash
$ kubectl create -f load-balancer-sticky/service-sticky-lb.yaml
```


После установки манифеста
```bash
$ watch kubectl get service
NAME                 CLUSTER-IP     EXTERNAL-IP       PORT(S)        AGE
nginx-internal-lb   10.0.0.10      192.168.0.181     80:30000/TCP   5m
``` 