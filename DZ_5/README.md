Домашнее задание: Сетевое взаимодействие в Kubernetes. Часть 

## Цель работы
Научиться организовывать взаимодействие между приложениями (frontend и backend) внутри кластера с помощью Service, а также настраивать внешний доступ через Ingress с правилами маршрутизации на основе пути запроса.

---

## Задание 1. Создание Deployment и Service

### 1.1. Развертывание приложений
- Создан Deployment `frontend-deployment` с 3 репликами контейнера nginx.
- Создан Deployment `backend-deployment` с 1 репликой контейнера multitool.
- Созданы Service `frontend-service` и `backend-service` для доступа к приложениям внутри кластера.

**Манифесты:**
- [deployment-frontend.yaml](deployment-frontend.yaml)
- [deployment-backend.yaml](deployment-backend.yaml)
- [service-frontend.yaml](service-frontend.yaml)
- [service-backend.yaml](service-backend.yaml)

### 1.2. Проверка взаимодействия
С помощью тестового пода `multitool-client` (создан в предыдущих ДЗ) выполнены запросы к сервисам:

**Запрос к frontend:**
```
kubectl exec multitool-client -- curl http://frontend-service
Результат: стандартная страница nginx (файл frontend-response.txt)

Запрос к backend:

kubectl exec multitool-client -- curl http://backend-service
Результат: ответ от multitool с информацией о поде (файл backend-response.txt)

Скриншоты выполнения:
рис.1_02,рис. 1_03, рис.1_05

Задание 2. Настройка Ingress
2.1. Включение Ingress-controller

microk8s enable ingress
kubectl get pods -n ingress
рис.2_03

2.2. Создание Ingress
Создан Ingress main-ingress с правилами:

Запросы на / → frontend-service (nginx)

Запросы на /api → backend-service (multitool)

Манифест: ingress.yaml

kubectl apply -f ingress.yaml
kubectl get ingress
рис.2_03

2.3. Проверка внешнего доступа
С локальной машины выполнены запросы к внешнему IP кластера (93.77.189.38):

Запрос к корневому пути:

curl http://93.77.189.38
Результат: страница nginx (файл frontend-external.txt)
рис.2_03

Запрос к пути /api:

curl http://93.77.189.38/api
Результат: ответ от backend (файл backend-external.txt)
рис.2_03

Выводы
В ходе работы были успешно решены следующие задачи:

Созданы многокомпонентные приложения (frontend и backend) с использованием Deployment.

Настроены Service для обнаружения и взаимодействия приложений внутри кластера.

Включен Ingress-controller и создан Ingress с правилами маршрутизации на основе пути.

Проверена доступность приложений как внутри кластера, так и снаружи.
