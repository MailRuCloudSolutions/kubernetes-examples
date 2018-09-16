# Поддержка WebSocket

Для того, чтобы активировать балансировке веб-сокетов в NGINX Ingress Controller, вам необходимо добавить аннотацию **nginx.org/websocket-services** к вашему Ingress-ресурсу. 
Данная аннотация определяет какие сервисы работают по протоколу веб-сокетов. У этой аннотации следующий синтаксис::
```
nginx.org/websocket-services: "service1[,service2,...]"
```

Следующий пример показывает как осуществляется балансировка между 3 сервисами, 1 из которых работает только на WebSocket:

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    nginx.org/websocket-services: "ws-svc"
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
      - path: /ws
        backend:
          serviceName: ws-svc
          servicePort: 8008
```