# Возможности конфигурирования предустановленного NGINX Ingress Controller

Вы можете управлять различными опциями NGINX Ingress Controller с помощью ConfigMap или аннотаций.
В случае использования ConfigMap, данные параметры будут применены глобально ко всем ресурсам Ingress, в случае аннотаций - только к тому Ingress, в котором вы использовали эту аннотацию.
Следующая таблица показывает соответствие между доступными аннотациями и ключами ConfigMap.
> Важно, что определенные параметры могут быть сконфигурированы только с помощью ConfigMap, либо только с помощью аннотаций.  



| Аннотация | Ключ ConfigMap | Описание| Значение по умолчанию |
| ---------- | -------------- | ----------- | ------- | 
| `kubernetes.io/ingress.class` | N/A | Specifies which Ingress controller must handle the Ingress resource. Set to `nginx` to make NGINX Ingress controller handle it. | N/A |
| `nginx.org/proxy-connect-timeout` | `proxy-connect-timeout` | Sets the value of the [proxy_connect_timeout](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_connect_timeout) directive. | `60s` | 
| `nginx.org/proxy-read-timeout` | `proxy-read-timeout` | Sets the value of the [proxy_read_timeout](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_read_timeout) directive. | `60s` | 
| `nginx.org/client-max-body-size` | `client-max-body-size` | Sets the value of the [client_max_body_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size) directive. | `1m` | 
| `nginx.org/proxy-buffering` | `proxy-buffering` | Enables or disables [buffering of responses](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_buffering) from the proxied server. | `True` | 
| `nginx.org/proxy-buffers` | `proxy-buffers` | Sets the value of the [proxy_buffers](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_buffers) directive. | Depends on the platform. | 
| `nginx.org/proxy-buffer-size` | `proxy-buffer-size` | Sets the value of the [proxy_buffer_size](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_buffer_size) directive | Depends on the platform. | 
| `nginx.org/proxy-max-temp-file-size` | `proxy-max-temp-file-size` | Sets the value of the  [proxy_max_temp_file_size](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_max_temp_file_size) directive. | `1024m` | 
| `nginx.org/proxy-hide-headers` | `proxy-hide-headers` | Sets the value of one or more  [proxy_hide_header](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_hide_header) directives. Example: `"nginx.org/proxy-hide-headers": "header-a,header-b"` | N/A | 
| `nginx.org/proxy-pass-headers` | `proxy-pass-headers` | Sets the value of one or more   [proxy_pass_header](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass_header) directives. Example: `"nginx.org/proxy-pass-headers": "header-a,header-b"` | N/A | 
| N/A | `server-names-hash-bucket-size` | Sets the value of the [server_names_hash_bucket_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_bucket_size) directive. | Depends on the size of the processor’s cache line. | 
| N/A | `server-names-hash-max-size` | Sets the value of the [server_names_hash_max_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_max_size) directive. | `512` | 
| N/A | `http2` | Enables HTTP/2 in servers with SSL enabled. | `False` |
| `nginx.org/redirect-to-https` | `redirect-to-https` | Sets the 301 redirect rule based on the value of the `http_x_forwarded_proto` header on the server block to force incoming traffic to be over HTTPS. Useful when terminating SSL in a load balancer in front of the Ingress controller — see [115](https://github.com/nginxinc/kubernetes-ingress/issues/115) | `False` | 
| `ingress.kubernetes.io/ssl-redirect` | `ssl-redirect` | Sets an unconditional 301 redirect rule for all incoming HTTP traffic to force incoming traffic over HTTPS. | `True` | 
| N/A | `log-format` | Sets the custom [log format](http://nginx.org/en/docs/http/ngx_http_log_module.html#log_format).  | See the [template file](../../nginx-controller/nginx/nginx.conf.tmpl). | 
| `nginx.org/hsts` | `hsts` | Enables [HTTP Strict Transport Security (HSTS)](https://www.nginx.com/blog/http-strict-transport-security-hsts-and-nginx/): the HSTS header is added to the responses from backends. The `preload` directive is included in the header. | `False` | 
| `nginx.org/hsts-max-age` | `hsts-max-age` | Sets the value of the `max-age` directive of the HSTS header. | `2592000` (1 month) |
| `nginx.org/hsts-include-subdomains` | `hsts-include-subdomains` | Adds the `includeSubDomains` directive to the HSTS header. | `False`| 
| N/A | `ssl-protocols` | Sets the value of the [ssl_protocols](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols) directive. | `TLSv1 TLSv1.1 TLSv1.2`| 
| N/A | `ssl-prefer-server-ciphers` | Enables or disables the [ssl_prefer_server_ciphers](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_prefer_server_ciphers) directive. | `False`| 
| N/A | `ssl-ciphers` | Sets the value of the [ssl_ciphers](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_ciphers) directive. | `HIGH:!aNULL:!MD5`| 
| N/A | `ssl-dhparam-file` | Sets the content of the dhparam file. The controller will create the file and set the value of the [ssl_dhparam](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_dhparam) directive with the path of the file.  | N/A | 
| N/A | `set-real-ip-from` | Sets the value of the [set_real_ip_from](http://nginx.org/en/docs/http/ngx_http_realip_module.html#set_real_ip_from) directive. | N/A | 
| N/A | `real-ip-header` | Sets the value of the [real_ip_header](http://nginx.org/en/docs/http/ngx_http_realip_module.html#real_ip_header) directive. | `X-Real-IP`| 
| N/A | `real-ip-recursive` | Enables or disables the [real_ip_recursive](http://nginx.org/en/docs/http/ngx_http_realip_module.html#real_ip_recursive) directive. | `False`| 
| `nginx.org/server-tokens` | `server-tokens` | Enables or disables the [server_tokens](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_tokens) directive. Additionally, with the NGINX Plus, you can specify a custom string value, including the empty string value, which disables the emission of the “Server” field. | `True`| 
| N/A | `main-snippets` | Sets a custom snippet in main context. | N/A | 
| N/A | `http-snippets` | Sets a custom snippet in http context. | N/A | 
| `nginx.org/location-snippets` | `location-snippets` | Sets a custom snippet in location context. | N/A | 
| `nginx.org/server-snippets` | `server-snippets` | Sets a custom snippet in server context. | N/A | 
| `nginx.org/lb-method` | `lb-method` | Sets the [load balancing method](https://www.nginx.com/resources/admin-guide/load-balancer/#method). The default `""` specifies the round-robin method. | `""` | 
| `nginx.org/listen-ports` | N/A | Configures HTTP ports that NGINX will listen on. | `[80]` | 
| `nginx.org/listen-ports-ssl` | N/A | Configures HTTPS ports that NGINX will listen on. | `[443]` | 
| N/A | `worker-processes` | Sets the value of the [worker_processes](http://nginx.org/en/docs/ngx_core_module.html#worker_processes) directive. | `auto` | 
| N/A | `worker-rlimit-nofile` | Sets the value of the [worker_rlimit_nofile](http://nginx.org/en/docs/ngx_core_module.html#worker_rlimit_nofile) directive. | N/A | 
| N/A | `worker-connections` | Sets the value of the [worker_connections](http://nginx.org/en/docs/ngx_core_module.html#worker_connections) directive. | `1024` | 
| N/A | `worker-cpu-affinity` | Sets the value of the [worker_cpu_affinity](http://nginx.org/en/docs/ngx_core_module.html#worker_cpu_affinity) directive. | N/A | 
| N/A | `worker-shutdown-timeout` | Sets the value of the [worker_shutdown_timeout](http://nginx.org/en/docs/ngx_core_module.html#worker_shutdown_timeout) directive. | N/A | 
| `nginx.org/keepalive` | `keepalive` | Sets the value of the [keepalive](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive) directive. Note that `proxy_set_header Connection "";` is added to the generated configuration when the value > 0. | `0` | 
| N/A | `proxy-protocol` | Enables PROXY Protocol for incoming connections. | `False` | 
| `nginx.org/rewrites` | N/A | Configures URI rewriting. | N/A | 
| `nginx.org/ssl-services` | N/A | Enables HTTPS when connecting to the endpoints of services. | N/A | 
| `nginx.org/websocket-services` | N/A | Enables WebSocket for services. | N/A |
| `nginx.org/max-fails` | `max-fails` | Sets the value of the [max_fails](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#max_fails) parameter of the `server` directive. | `1` | 
| `nginx.org/fail-timeout` | `fail-timeout` | Sets the value of the [fail_timeout](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#fail_timeout) parameter of the `server` directive. | `10s` | 

## Конфигурация с помощью ConfigMaps
1. Отредактируйте файл `nginx-config.yaml`, установив необходимые вам параметры.
2. Примените данный файл на вашем кластере Kubernetes:
    ```
    $ kubectl apply -f nginx-config.yaml
    ```
    После этого, конфигурация NGINX Ingress Controller будет изменена.

3. Если вам необходимо обновить некоторые параметры, измените файл `nginx-config.yaml` и выполните следующую команду еще раз:
    ```
    $ kubectl apply -f nginx-config.yaml
    ```
    

## Конфигурация с помощью аннотаций

Если вам необходимо настроить параметры для конкретного Ingress, проще всего использовать аннотации.
Значения, используемые в аннотациях имеют больший приоритет, чем ConfigMap.

Например (**cafe-ingress-with-annotations.yaml**):
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: cafe-ingress-with-annotations
  annotations:
    nginx.org/proxy-connect-timeout: "30s"
    nginx.org/proxy-read-timeout: "20s"
    nginx.org/client-max-body-size: "4m"
    nginx.org/location-snippets: |
        if ($ssl_client_verify = SUCCESS) {
            set $auth_basic off;
        }
        if ($ssl_client_verify != SUCCESS) {
            set $auth_basic "Restricted";
        }
        auth_basic $auth_basic;
        auth_basic_user_file "/var/run/secrets/nginx.org/auth-basic-file";
    nginx.org/server-snippets: |
        ssl_verify_client optional;
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
```
Annotations take precedence over ConfigMaps.
