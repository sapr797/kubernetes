# Домашнее задание: Сетевое взаимодействие в Kubernetes

## Задание 1. Service ClusterIP и NodePort

### 1.1. Deployment с двумя контейнерами
[deployment-multi-container.yaml](deployment-multi-container.yaml)

### 1.2. Service ClusterIP (многопортовый)
[service-clusterip.yaml](service-clusterip.yaml)

### 1.3. Проверка доступа внутри кластера
```
kubectl run test-pod --image=wbitt/network-multitool -it --rm --restart=Never -- curl multi-app-service:9001
Вывод (nginx):

<!DOCTYPE html>...
Сохранён в check-clusterip-nginx.txt.

kubectl run test-pod --image=wbitt/network-multitool -it --rm --restart=Never -- curl multi-app-service:9002
Вывод (multitool):

WBITT Network MultiTool...
Сохранён в check-clusterip-multitool.txt.

1.4. Service NodePort для внешнего доступа
service-nodeport.yaml

Проверка с локальной машины:

curl http://84.201.172.206:30080
Вывод (страница nginx) сохранён в check-nodeport.txt.
рис.1_04+. рис.1_00-рис.1_09

Задание 2. Ingress
2.1. Развертывание frontend и backend
deployment-frontend.yaml

deployment-backend.yaml

service-frontend.yaml

service-backend.yaml

2.2. Ingress с правилами маршрутизации
ingress.yaml

2.3. Проверка доступности

curl http://myapp.local
Вывод (nginx) сохранён в check-ingress-root.txt.

bash
curl http://myapp.local.api
Вывод (multitool) сохранён в check-ingress-api.txt.
рис.2_01, рис.2_02,  рис.2_03

Выводы
В ходе работы успешно настроены:

многоконтейнерный Deployment,

многопортовый ClusterIP Service,

NodePort для внешнего доступа,

Ingress с разделением трафика по путям.

Все манифесты и подтверждения работы находятся в данной папке.
