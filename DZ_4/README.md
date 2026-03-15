# Домашнее задание №4: Сетевое взаимодействие в Kubernetes (часть 1)

## Цель работы
Освоить механизмы сетевого взаимодействия в Kubernetes: создание многопортовых Service для доступа к разным контейнерам внутри пода, балансировка нагрузки между репликами, а также публикация сервисов наружу с помощью NodePort.

---

## Задание 1. Внутрикластерный доступ к разным контейнерам

### 1.1. Создание Deployment с тремя репликами
Был создан Deployment `nginx-multitool-deploy`, который запускает 3 пода, каждый с двумя контейнерами:
- `nginx` (слушает порт 80)
- `multitool` (настроен на порт 8080 через переменную окружения `HTTP_PORT`, чтобы избежать конфликта с nginx)

**Манифест:** [`deployment-3replicas.yaml`](deployment-3replicas.yaml)

```
kubectl apply -f deployment-3replicas.yaml
Результат:
kubectl get pods
рис. 1_03 или текстовый файл pods.txt

1.2. Создание Service (ClusterIP)
Создан Service multi-app-service типа ClusterIP, который направляет трафик:

с порта 9001 на порт 80 контейнера nginx

с порта 9002 на порт 8080 контейнера multitool

Манифест: service-clusterip.yaml

kubectl apply -f service-clusterip.yaml
Проверка сервиса и эндпоинтов:

kubectl get svc multi-app-service
kubectl get endpoints multi-app-service
рис. 1_05 или файлы service.txt, endpoints.txt

1.3. Проверка доступа из отдельного пода
Запущен Pod multitool-client для выполнения тестовых запросов.

Манифест: client-pod.yaml

kubectl apply -f client-pod.yaml
Из клиентского пода выполнены запросы к сервису:

Доступ к nginx через порт 9001:

kubectl exec multitool-client -- curl multi-app-service:9001
рис.1_05  или файл curl-nginx.txt

Доступ к multitool через порт 9002

kubectl exec multitool-client -- curl multi-app-service:9002
При повторных запросах имя пода в ответе менялось, что подтверждает балансировку нагрузки между тремя репликами.
рис.1_05 или файл curl-multitool.txt

Задание 2. Внешний доступ через NodePort
2.1. Создание Service типа NodePort
Создан отдельный Service nginx-nodeport типа NodePort, который открывает доступ к контейнерам nginx (порт 80) снаружи кластера. Использован фиксированный порт узла 30080.

Манифест: service-nodeport.yaml

kubectl apply -f service-nodeport.yaml
Проверка сервиса:

kubectl get svc nginx-nodeport
рис.2_02

2.2. Проверка доступа с локального компьютера
С локальной машины (вне кластера) выполнен запрос к узлу кластера по внешнему IP 93.77.189.38 на порт 30080:

curl http://93.77.189.38:30080
Результат — стандартная страница nginx.
рис.2_03 или файл curl-nodeport.txt

Также доступ можно открыть в браузере:

http://93.77.189.38:30080/
Файлы манифестов
deployment-3replicas.yaml

service-clusterip.yaml

client-pod.yaml

service-nodeport.yaml

Файлы с выводами команд (опционально)
pods.txt

service.txt

endpoints.txt

curl-nginx.txt

curl-multitool.txt

nodeport-service.txt

curl-nodeport.txt

Все перечисленные файлы находятся в данной папке.

Выводы
В ходе работы были успешно решены следующие задачи:

создан Deployment с тремя репликами, каждая из которых содержит два контейнера;

настроен Service, обеспечивающий доступ к разным контейнерам внутри кластера по разным портам;

проверена балансировка нагрузки между репликами;

реализован внешний доступ к приложению через NodePort;

проведена валидация с локального компьютера.
