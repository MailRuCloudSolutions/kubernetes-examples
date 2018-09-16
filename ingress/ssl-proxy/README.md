# Поддержка проксирования SSL-трафика

NGINX Ingress Controller поддерживает различные способы управления трафиком, в том числе если терминация SSL-трафика происходит не на уровне Ingress, а напрямую в конкретных сервисах. Т.е. это проксирование SSL-трафика.  

Для того, чтобы активировать эту возможность вам нужно добавить аннотацию **nginx.org/ssl-services** к декларации Ingress. Данная аннотация указывает на какие сервисы нужно проксировать SSL-трафик, не терминируя его на уровне Ingress Controller. У этой аннотации следующий синтаксис:
```
nginx.org/ssl-services: "service1[,service2,...]"
```

Следующий пример показывает как осуществляется балансировка между 3 сервисами, 1 из которых работает только на HTTPS:
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    nginx.org/ssl-services: "ssl-svc"
spec:
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /tea
        backend:
          serviceName: tea-svc
          servicePort: 80
      - path: /coffee
        backend:
          serviceName: coffee-svc
          servicePort: 80
      - path: /ssl
        backend:
          serviceName: ssl-svc
          servicePort: 443
```
