## Домашнее задание по теме "Настройка приложений и управление доступом в K8s" ##  

### Задание 1 - Работа с configMaps ###  

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

4. 
