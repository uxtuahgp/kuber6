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
2.
