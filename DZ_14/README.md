# Домашнее задание: Диагностика и устранение неисправности

## Цель работы
Научиться выявлять и исправлять ошибки сетевого взаимодействия между компонентами приложения, развёрнутыми в разных namespace Kubernetes.

---

## 1. Исходный манифест
Был применён манифест из [task.yaml](https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml), который создаёт:
- Deployment `web-consumer` в namespace `web`
- Deployment `auth-db` в namespace `data`
- Service `auth-db` в namespace `data`

## 2. Выявление проблемы

После применения манифеста проверены поды:
```
kubectl get pods -n web
kubectl get pods -n data
Все поды запущены, но web-consumer перезапускался. Логи показали ошибку:

kubectl logs -n web -l app=web-consumer
Вывод (сохранён в logs-initial.txt):

curl: (6) Could not resolve host: auth-db
Причина: под в namespace web пытается обратиться к сервису auth-db по короткому имени, но сервис находится в другом namespace (data). В Kubernetes DNS-имена имеют вид <service>.<namespace>.svc.cluster.local.
рис.1_00, рис.1_01, рис.1_03, рис.1_05

3. Исправление
3.1. Корректировка команды в Deployment
Исправлен манифест web-consumer-fixed.yaml – в команде curl указано полное DNS-имя:

command: ["sh", "-c", "while true; do curl auth-db.data.svc.cluster.local; sleep 5; done"]
web-consumer-fixed.yaml
рис.1_05, рис.1_06

3.2. Применение исправления

kubectl apply -f web-consumer-fixed.yaml

3.3. Проверка

kubectl logs -n web -l app=web-consumer
Теперь логи содержат успешные ответы от nginx (сохранено в logs-fixed.txt):

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
рис.1_07,  рис.1_08
...
4. Выводы
Проблема была решена путём использования полного DNS-имени сервиса из другого namespace. Альтернативные варианты (объединение namespace или создание прокси-сервиса) не потребовались.
