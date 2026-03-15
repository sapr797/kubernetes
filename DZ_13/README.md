# Домашнее задание: Обновление приложений в Kubernetes

## Задание 1. Выбор стратегии обновления

**Условия:**
- Несколько реплик, мажорное обновление.
- Новые версии несовместимы со старыми.
- Ресурсы ограничены, запас 20% в часы низкой нагрузки.

**Рассмотренные стратегии:**
- RollingUpdate – требует дополнительных ресурсов и допускает сосуществование версий.
- Blue/Green – требует удвоения ресурсов.
- Canary – требует сосуществования версий.
- Recreate – завершает старые поды перед запуском новых, не требует доп. ресурсов.

**Выбор:** стратегия **Recreate**, так как она единственная полностью удовлетворяет условиям (несовместимость и ограниченность ресурсов).

*Примечание:* для минимизации downtime можно настроить RollingUpdate с maxSurge=0 и maxUnavailable=1, но это потребует совместимости версий. При невозможности сосуществования остаётся Recreate.
рис.1_00, рис.1_03 
---

## Задание 2. Практическое обновление приложения

### 2.1. Исходный Deployment
Создан Deployment с образами nginx:1.19 и multitool, 5 реплик.
[deployment-nginx-1.19.yaml](task2/deployment-nginx-1.19.yaml)

### 2.2. Обновление до 1.20
Использована команда `kubectl set image`. Процесс отслеживался через `kubectl rollout status`. Приложение оставалось доступным (проверено через тестовый под).

### 2.3. Имитация неудачного обновления (1.28)
Попытка обновления до несуществующей версии привела к ошибке ImagePullBackOff.

### 2.4. Откат
Выполнен откат через `kubectl rollout undo`. Приложение восстановлено.

**Результаты:**
- История ревизий: [rollout-history.txt](task2/rollout-history.txt)
- Поды после отката: [pods-after-rollback.txt](task2/pods-after-rollback.txt)
- Статус Deployment: [deployment-status.txt](task2/deployment-status.txt)
- Проверка доступности nginx: [nginx-check.txt](task2/nginx-check.txt)
- Проверка доступности multitool: [multitool-check.txt](task2/multitool-check.txt)

![Скриншоты процесса] рис.2_00

---

## Задание 3*. Канареечное развертывание

### 3.1. Подготовка ConfigMap для двух версий
- [configmap-v1.yaml](task3/configmap-v1.yaml) – стабильная версия страницы
- [configmap-v2.yaml](task3/configmap-v2.yaml) – канареечная версия

### 3.2. Создание двух Deployment
- Стабильная (4 реплики): [deployment-stable.yaml](task3/deployment-stable.yaml)
- Канареечная (1 реплика): [deployment-canary.yaml](task3/deployment-canary.yaml)

### 3.3. Service и Ingress
- Общий Service: [service-canary.yaml](task3/service-canary.yaml)
- Ingress с весом 20% на canary: [ingress-canary.yaml](task3/ingress-canary.yaml)

### 3.4. Проверка
Выполнено 10 запросов. Результаты показывают, что примерно 20% трафика направляется на канареечную версию.
$ for i in {1..10}; do curl -s http://canary.example.com | grep "Welcome"; done
Welcome to Stable Nginx (v1)
Welcome to Stable Nginx (v1)
Welcome to Canary Nginx (v2)
Welcome to Stable Nginx (v1)
Welcome to Stable Nginx (v1)
Welcome to Canary Nginx (v2)
Welcome to Stable Nginx (v1)
Welcome to Stable Nginx (v1)
Welcome to Stable Nginx (v1)
Welcome to Canary Nginx (v2)


Файлы:
- Поды: [canary-pods.txt](task3/canary-pods.txt)
- Описание Ingress: [canary-ingress.txt](task3/canary-ingress.txt)
- Пример ответа: [canary-response.txt](task3/canary-response.txt)

![Скриншоты](рис.3_01-рис.3_05)

---

## Выводы
В ходе работы:
- Проанализированы стратегии обновления и выбрана оптимальная для заданных условий.
- Выполнено практическое обновление приложения, имитация сбоя и откат.
- Реализовано канареечное развертывание с распределением трафика через Ingress.

Все манифесты и подтверждающие файлы находятся в соответствующих папках.
