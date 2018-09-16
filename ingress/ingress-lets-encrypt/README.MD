Наряду с применением заранее сгенерированных SSL-сертификатов для терминации TLS-трафика в Ingress, существует возможность использования бесплатных сертификатов, создаваемых при помощи сервиса Let's Encrypt. 

Данная инструкция описывает процесс активации Let's Encrypt для Ingress Controller на базе Nginx, который уже предустановлен в вашем кластере Kubernetes версиий 1.10 и 1.11.


1. Используя Helm установить cert-manager следующей командой:

```
helm install --name cert-manager --namespace kube-system stable/cert-manager --set ingressShim.defaultIssuerName=letsencrypt-prod --set ingressShim.defaultIssuerKind=ClusterIssuer
```

2. Получите внешний IP-адрес nginx-ingress контроллера этой командой

```
kubectl get svc -n ingress-nginx| grep ingress | grep nginx | grep LoadBalancer
```
3. Заделегируйте A-запись вашего домена на внешний IP-адрес Ingress Controller

4. Установите ClusterIssuers, предварительно заменив значение поля email на ваш реальный email

```
kubectl apply -f cluster-issuers.yaml
```


```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: your@mail.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable HTTP01 validations
    http01: {}
---
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # The ACME server URL
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: your@mail.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-staging
    # Enable the HTTP-01 challenge provider
    http01: {}

```

Cluster Issuer letsencrypt-prod подходит для production-сертификатов и требует обязательной делегации домена на внешний IP-адрес Ingress Controller.
Сluster Issuer letsencrypt-staging подходит только для разработки и не требует реальной делегации домена. Однако, выпущенные им сертификаты не подойдут для production использования.

5. Создайте целевое приложение и сервис. Пример ниже создает Replication Controller на базе nginx и ассоциированный с ним сервис

```
kubectl apply -f service-rc.yaml
```

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 2
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
---
apiVersion: v1
kind: Service
metadata:
  labels:
    name: nginx
  name: nginx
spec:
  ports:
    # The port that this service should serve on.
    - port: 80
  # Label keys and values that must match in order to receive traffic for this service.
  selector:
    app: nginx
```

6. Создайте Ingress

6.1. В случае production используйте данный манифест:

```
kubectl apply -f ingress-production.yaml
```

В нем нужно заменить production.example.com на ваше доменное имя.
Также обратите внимание на аннотацию ingress.kubernetes.io/ssl-redirect: "true", которая делает обязательный редирект с http на https


```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: production-ingress
  annotations:
    ingress.kubernetes.io/ssl-redirect: "true"
    kubernetes.io/tls-acme: "true"
    certmanager.k8s.io/cluster-issuer: letsencrypt-prod
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - production.example.com
    secretName: letsencrypt-cluster-prod
  rules:
  - host: production.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx
          servicePort: 80

```

Важно: 
Если вам необходимо использовать Let's Encrypt для нескольких доменов (не только 2го уровня), то следует придерживаться следующих правил:
1) Для каждого домена нужно создавать отдельную сущность Ingress
2) Для каждого домена нужно создать выделенный Issuer или ClusterIssuer.
3) Каждый Issuer/ClusterIssuer должен использовать уникальное имя Secret.


7.2. В случае необходимости сгенерировать сертификат для разработки используйте следующий манифест:

```
kubectl apply -f ingress-staging.yaml
```

В нем staging.example.com нужно заменить на ваш домен.

```yaml

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: staging-ingress
  annotations:
    ingress.kubernetes.io/ssl-redirect: "true"
    kubernetes.io/tls-acme: "true"
    certmanager.k8s.io/cluster-issuer: letsencrypt-staging
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - staging.example.com
    secretName: letsencrypt-staging
  rules:
  - host: staging.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx
          servicePort: 80

```


#Для отладки используйте следующие команды:

1. Получение информации по Ingress

В случае успешной генерации Let's Encrypt сертификата, вы должны увидеть сообщение "letsencrypt-cluster-prod terminates production.example.com" в выводе этой команды.
```
kubectl describe ingress production-ingress
```


```yaml
Name:             production-ingress
Namespace:        default
Address:
Default backend:  default-http-backend:80 (<none>)
TLS:
  letsencrypt-cluster-prod terminates production.example.com
Rules:
  Host               Path  Backends
  ----               ----  --------
  production.example.com
                     /   nginx:80 (<none>)
Annotations:
Events:  <none>
```


2. Получение информации о сертификате 


В случае успешной линковки сертификата и Ingress вы должны увидеть строчку "Ingress:  production-ingress" в выводе команды

```
kubectl describe certificate letsencrypt-cluster-prod
```
 

```yaml
Name:         letsencrypt-cluster-prod
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  certmanager.k8s.io/v1alpha1
Kind:         Certificate
Metadata:
  Cluster Name:
  Creation Timestamp:  2018-06-04T19:07:13Z
  Owner References:
    API Version:           extensions/v1beta1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  Ingress
    Name:                  production-ingress
    UID:                   7d1823a8-682a-11e8-868e-fa163e8a2d54
  Resource Version:        584861
  Self Link:               /apis/certmanager.k8s.io/v1alpha1/namespaces/default/certificates/letsencrypt-cluster-prod
  UID:                     7d1b4ebf-682a-11e8-868e-fa163e8a2d54
Spec:
  Acme:
    Config:
      Domains:
        production.example.com
      Http 01:
        Ingress:  production-ingress
  Common Name:
  Dns Names:
    production.example.com
  Issuer Ref:
    Kind:       ClusterIssuer
    Name:       letsencrypt-prod
  Secret Name:  letsencrypt-cluster-prod
Status:
Events:  <none>
```

3. Получение логов генерации сертификатов cert-manager

Например, 
```yaml
kubectl logs -n kube-system cert-manager-cert-manager-bcb9887f-llm6x
```