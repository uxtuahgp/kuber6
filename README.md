## Домашнее задание по теме "Настройка приложений и управление доступом в K8s"

### Задание 1 - Работа с configMaps

1. Создал и применил [манифест configMap](configmap.yml) в котором сохранил содержимое HTML документа
2. Создал и применил [манифест Deployment](deployment.yml) контейнеры которого который через volume получают содержимое документа
3. Проверил доступность страницы из одного из Pod

```
alex@uxtu-note:~/Study/kuber6/kuber6$ kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
cm-app-7d6f7f7695-vkb7l   2/2     Running   0          6s
alex@uxtu-note:~/Study/kuber6/kuber6$ kubectl exec -it cm-app-7d6f7f7695-vkb7l --container nginx -- bash
root@cm-app-7d6f7f7695-vkb7l:/# curl localhost/demo.html
<html>
<head>configMap demo!</head>
<body>
U popa byla
sobaka On ee lubil
Ona syela kusok myasa
On ee lubil
</body>
</html>
root@cm-app-7d6f7f7695-vkb7l:/# curl localhost:8080/demo.html
<html>
<head>configMap demo!</head>
<body>
U popa byla
sobaka On ee lubil
Ona syela kusok myasa
On ee lubil
</body>
</html>
```

4. Для собственного развития и повторения создал и применил [сервис](svc.yml) и [ингрес](ingress.yml) для доступа к документу с локального хоста.

```
alex@uxtu-note:~/Study/kuber6/kuber6$ curl localhost/nginx/demo.html
<html>
<head>configMap demo!</head>
<body>
U popa byla
sobaka On ee lubil
Ona syela kusok myasa
On ee lubil
</body>
</html>
alex@uxtu-note:~/Study/kuber6/kuber6$ curl localhost/nginx
<html>
<head>configMap demo!</head>
<body>
U popa byla
sobaka On ee lubil
Ona syela kusok myasa
On ee lubil
</body>
</html>
alex@uxtu-note:~/Study/kuber6/kuber6$ curl localhost/mtool
<html>
<head>configMap demo!</head>
<body>
U popa byla
sobaka On ee lubil
Ona syela kusok myasa
On ee lubil
</body>
</html>
```

### Задание 2 - работа с Secrets

1. Сгенерировал самоподписанный сертификат и ключ и закодировал их base64 для передачи в качестве секрета

```
alex@uxtu-note:~/Study/kuber6/kuber6/task2$ openssl req -x509 -nodes -days 365 -newkey rsa:2048   -keyout tls.key -out tls.crt -subj "/CN=myapp.example.com"
.+..+......+.+........+................+.....+.+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*........+....+..+.......+...+..+.......+.....++++++++++++++++++++++++++++++++++++++++++++++
...
alex@uxtu-note:~/Study/kuber6/kuber6/task2$ base64 < tls.crt -w0
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURHVENDQWdHZ0F3SUJBZ0lVV1VCZ3hSZHZCTjc3d...
alex@uxtu-note:~/Study/kuber6/kuber6/task2$ base64 < tls.key -w0
LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2UUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQ...
```

2. Создал и применил [манифест секрета](task2/secret.yml)

```
lex@uxtu-note:~/Study/kuber6/kuber6/task2$ kubectl apply -f secret.yml
secret/my-sec created
```

3. Развернул [deployment](task2/deployment.yml) со своим [configMap](task2/configmap.yml) для проброса demo.html
4. Создал [service](task2/svc.yml) и [ingress](task2/ingress.yml) с применением tls секрета

```
alex@uxtu-note:~/Study/kuber6/kuber6/task2$ kubectl apply -f svc.yml
service/sec-svc created
alex@uxtu-note:~/Study/kuber6/kuber6/task2$ kubectl apply -f ingress.yml
ingress.networking.k8s.io/cm-ingress created
alex@uxtu-note:~/Study/kuber6/kuber6/task2$ curl https://localhost
curl: (60) SSL certificate problem: self-signed certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
alex@uxtu-note:~/Study/kuber6/kuber6/task2$ curl -k https://localhost/demo.html
<html>
<head>configMap demo!</head>
<body>
U popa byla
sobaka On ee lubil
Ona syela kusok myasa
On ee lubil
</body>
</html>
```

### Задание 3 - работа с RBAC

1. Включил RBAC в microk8s

```
alex@uxtu-note:~/Study/kuber6/kuber6/task3$ microk8s enable rbac
Infer repository core for addon rbac
Enabling RBAC
Reconfiguring apiserver
Restarting apiserver
RBAC is enabled
```

2. Создал сертификат для пользователя pupkin

```
alex@uxtu-note:~/Study/kuber6/kuber6/task3$ openssl genrsa -out developer.key 2048
alex@uxtu-note:~/Study/kuber6/kuber6/task3$ openssl req -new -key developer.key -out developer.csr -subj "/CN=pupkin"
alex@uxtu-note:~/Study/kuber6/kuber6/task3$ ls -l /var/snap/microk8s/current/certs/
итого 112
-rw-rw---- 1 root microk8s 1062 апр 27 23:29 apiserver-kubelet-client.crt
-rw-rw---- 1 root microk8s 1675 апр 27 23:02 apiserver-kubelet-client.key
-rw-r--r-- 1 root root     1123 апр 27 23:29 ca.crt
-rw-rw---- 1 root microk8s 1679 апр 27 23:02 ca.key
-rw-rw---- 1 root microk8s   41 мая 13 21:01 ca.srl
-rw-rw---- 1 root microk8s 1029 апр 27 23:29 client.crt
-rw-rw---- 1 root microk8s 1675 апр 27 23:02 client.key
-rw-rw---- 1 root microk8s 1029 апр 27 23:29 controller.crt
-rw-rw---- 1 root microk8s 1679 апр 27 23:02 controller.key
-rw-r--r-- 1 root root      770 мая 13 21:01 csr.conf
-rw-rw---- 1 root microk8s  770 мая 13 23:42 csr.conf.rendered
-rw-rw---- 1 root microk8s  704 апр 27 23:29 csr.conf.template
-rw-rw---- 1 root microk8s 1127 апр 27 23:02 front-proxy-ca.crt
-rw-rw---- 1 root microk8s 1679 апр 27 23:02 front-proxy-ca.key
-rw-rw---- 1 root microk8s   41 мая 13 21:01 front-proxy-ca.srl
-rw-r--r-- 1 root root     1480 мая 13 21:01 front-proxy-client.crt
-rw-rw---- 1 root microk8s 1192 мая 13 21:01 front-proxy-client.csr
-rw-rw---- 1 root microk8s 1675 апр 27 23:02 front-proxy-client.key
-rw-rw---- 1 root microk8s 1119 апр 27 23:29 kubelet.crt
-rw-rw---- 1 root microk8s 1675 апр 27 23:02 kubelet.key
-rw-rw---- 1 root microk8s 1013 апр 27 23:29 proxy.crt
-rw-rw---- 1 root microk8s 1679 апр 27 23:02 proxy.key
-rw-rw---- 1 root microk8s 1017 апр 27 23:29 scheduler.crt
-rw-rw---- 1 root microk8s 1675 апр 27 23:02 scheduler.key
-rw-r--r-- 1 root root     1590 мая 13 21:01 server.crt
-rw-rw---- 1 root microk8s 1305 мая 13 21:01 server.csr
-rw-rw---- 1 root microk8s 1675 апр 27 23:02 server.key
-rw-rw---- 1 root microk8s 1675 апр 27 23:02 serviceaccount.key
alex@uxtu-note:~/Study/kuber6/kuber6/task3$ openssl x509 -req -in developer.csr -CA /var/snap/microk8s/current/certs/ca.crt -CAkey /var/snap/microk8s/current/certs/ca.key -CAcreateserial -out developer.crt -days 365
Certificate request self-signature ok
subject=CN = pupkin
```

3. Создал пользователя и контекст пользователя

```
alex@uxtu-note:~/Study/kuber6/kuber6/task3$ kubectl config set-credentials pupkin --client-certificate developer.crt --client-key developer.key
User "pupkin" set.
alex@uxtu-note:~/Study/kuber6/kuber6/task3$ kubectl config set-context pupkin-context --cluster microk8s-cluster --user pupkin
Context "pupkin-context" created.

```

4. Создал системного пользователя и из своего конфига config для пользователя pupkin

```
alex@uxtu-note:~/Study/kuber6/kuber6/task3$ sudo ls -l /home/pupkin/.kube
итого 16
-rw-r--r-- 1 pupkin pupkin 1922 мая 14 00:03 config
-rw-r--r-- 1 pupkin pupkin  993 мая 14 00:05 developer.crt
-rw-r--r-- 1 pupkin pupkin  887 мая 14 00:05 developer.csr
-rw------- 1 pupkin pupkin 1704 мая 14 00:05 developer.key
alex@uxtu-note:~/Study/kuber6/kuber6/task3$ sudo cat /home/pupkin/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0...
    server: https://192.168.0.100:16443
  name: microk8s-cluster
contexts:
- context:
    cluster: microk8s-cluster
    user: pupkin
  name: pupkin-context
current-context: pupkin-context
kind: Config
preferences: {}
users:
- name: pupkin
  user:
    client-certificate: /home/pupkin/.kube/developer.crt
    client-key: /home/pupkin/.kube/developer.key
```

5. Попробовал выполнить под пользователем элементарные команды

```
pupkin@uxtu-note:~$ kubectl config get-contexts
CURRENT   NAME             CLUSTER            AUTHINFO   NAMESPACE
*         pupkin-context   microk8s-cluster   pupkin
pupkin@uxtu-note:~$ kubectl get namespaces
Error from server (Forbidden): namespaces is forbidden: User "pupkin" cannot list resource "namespaces" in API group "" at the cluster scope
```

6. Создал и применил манифесты создания [role](task3/role.yml) и [rolebinding](task3/rolebindidng.yml)

```
alex@uxtu-note:~/Study/kuber6/kuber6/task3$ kubectl apply -f role.yml
role.rbac.authorization.k8s.io/my-role created
alex@uxtu-note:~/Study/kuber6/kuber6/task3$ kubectl apply -f rolebinding.yml
rolebinding.rbac.authorization.k8s.io/my-RoleBinding created
```

7. Проверил как работает примененная авторизация для пользователя pupkin

```
alex@uxtu-note:~/Study/kuber6/kuber6/task3$ kubectl get pods --as=pupkin
NAME                       READY   STATUS    RESTARTS   AGE
sec-app-85d4bf45d8-dghm7   1/1     Running   0          67m
```

```
pupkin@uxtu-note:~$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
sec-app-85d4bf45d8-dghm7   1/1     Running   0          59m
pupkin@uxtu-note:~$ kubectl logs sec-app-85d4bf45d8-dghm7
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/05/13 20:27:03 [notice] 1#1: using the "epoll" event method
2026/05/13 20:27:03 [notice] 1#1: nginx/1.31.0
2026/05/13 20:27:03 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
2026/05/13 20:27:03 [notice] 1#1: OS: Linux 6.8.0-110-generic
2026/05/13 20:27:03 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 65536:65536
2026/05/13 20:27:03 [notice] 1#1: start worker processes
2026/05/13 20:27:03 [notice] 1#1: start worker process 29
2026/05/13 20:27:03 [notice] 1#1: start worker process 30
2026/05/13 20:27:03 [notice] 1#1: start worker process 31
2026/05/13 20:27:03 [notice] 1#1: start worker process 32
10.1.69.227 - - [13/May/2026:20:27:10 +0000] "GET /demo.html HTTP/1.1" 200 124 "-" "curl/7.81.0" "192.168.0.100"
```
