# Домашнее задание №3: Запуск приложений в Kubernetes

## Цель работы
Научиться создавать Deployment с несколькими контейнерами, масштабировать приложения, организовывать доступ к репликам через Service, а также использовать Init-контейнеры для управления порядком запуска.

---

## Задание 1. Deployment с двумя контейнерами и доступ через Service

### 1.1. Создание Deployment
Был создан Deployment `nginx-multitool-deployment`, который запускает Pod с двумя контейнерами: `nginx` (порт 80) и `multitool` (порт 1180, изменён через переменные окружения для избежания конфликта).

**Манифест:** [`deployment-two-containers.yaml`](deployment-two-containers.yaml)

```bash
kubectl apply -f deployment-two-containers.yaml
1.2. Масштабирование до 2 реплик
Первоначально количество реплик было 1. После выполнения команды масштабирования количество реплик увеличено до 2.

До масштабирования:

kubectl get pods
рис.1_01 
После масштабирования:

kubectl scale deployment nginx-multitool-deployment --replicas=2
kubectl get pods
рис.1_02

1.3. Создание Service
Создан Service nginx-multitool-service, который обеспечивает доступ к обоим контейнерам внутри кластера:

порт 80 → nginx (targetPort 80)

порт 1180 → multitool (targetPort 1180)

Манифест: service.yaml

kubectl apply -f service.yaml
Проверка эндпоинтов:

kubectl get endpoints nginx-multitool-service
рис.1_02

1.4. Проверка доступа из отдельного пода
Запущен Pod multitool-client для выполнения запросов.

Манифест: pod-client.yaml

kubectl apply -f pod-client.yaml
Выполнены команды внутри клиентского пода:

Доступ к nginx:

kubectl exec -it multitool-client -- curl nginx-multitool-service:80
рис.1_02

Доступ к multitool:

kubectl exec -it multitool-client -- curl nginx-multitool-service:1180
рис.1_03

При повторных запросах к порту 1180 имя пода-обработчика менялось, подтверждая балансировку нагрузки между двумя репликами.

Задание 2. Deployment с Init-контейнером, ожидающим запуска Service
2.1. Создание Deployment с Init-контейнером
Создан Deployment nginx-init-deployment. Init-контейнер на основе busybox выполняет цикл с проверкой доступности сервиса nginx-init-service через nslookup. Основной контейнер nginx не запускается, пока init-контейнер не завершится успешно.

Манифест: deployment-with-init.yaml

kubectl apply -f deployment-with-init.yaml
2.2. Состояние пода до создания сервиса
На момент применения Deployment сервис nginx-init-service отсутствовал. Под перешёл в состояние Init:0/1.

bash
kubectl get pods -w
рис.2_03

Логи init-контейнера показывали ожидание:

kubectl logs nginx-init-deployment-xxx-xxx -c wait-for-service
рис.2_03

2.3. Создание Service
Создан Service nginx-init-service, который необходим для завершения init-контейнера.

Манифест: service-for-init.yaml

kubectl apply -f service-for-init.yaml
2.4. Состояние пода после создания сервиса
После создания сервиса init-контейнер успешно завершился, и основной контейнер nginx запустился. Под перешёл в состояние Running.

kubectl get pods
рис.2_03

Логи init-контейнера после завершения:

kubectl logs nginx-init-deployment-7457cdf6dd-btzvg -c wait-for-service
рис.2_02

2.5. Проверка работы nginx
Выполнен проброс порта и проверка доступности nginx:

kubectl port-forward pod/nginx-init-deployment-7457cdf6dd-btzvg 8080:80 &
curl http://localhost:8080

Файлы манифестов
deployment-two-containers.yaml
рис.2_03
service.yaml

pod-client.yaml

deployment-with-init.yaml

service-for-init.yaml

Все перечисленные файлы находятся в данной папке.

Выводы
В ходе выполнения работы были успешно решены следующие задачи:

создан Deployment с двумя контейнерами и решена проблема конфликта портов;

выполнено масштабирование приложения;

организован доступ к репликам через Service;

проверена балансировка нагрузки между репликами;

реализован Init-контейнер, ожидающий появления сервиса, что продемонстрировало управление порядком запуска в Kubernetes.
