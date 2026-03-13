# Домашнее задание №7: Сетевые политики в Kubernetes

## Цель работы
Научиться создавать и применять Network Policies для изоляции трафика между микросервисами. Необходимо обеспечить доступ только по цепочке frontend → backend → cache, запретив все остальные соединения.

---

## Задание 1. Развёртывание приложений и настройка политик

### 1.1. Создание namespace и deployment'ов
Создан namespace `app`. В нём развёрнуты три deployment'а (frontend, backend, cache) с образами `wbitt/network-multitool` и соответствующие сервисы.

**Манифесты:**
- [deployment-frontend.yaml](deployment-frontend.yaml)
- [deployment-backend.yaml](deployment-backend.yaml)
- [deployment-cache.yaml](deployment-cache.yaml)
- [test-pod.yaml](test-pod.yaml) – для тестирования

```
kubectl apply -f deployment-frontend.yaml
kubectl apply -f deployment-backend.yaml
kubectl apply -f deployment-cache.yaml
kubectl apply -f test-pod.yaml
Проверка запуска:

kubectl get pods -n app
рис.1_00, рис.1_01

1.2. Проверка доступности до применения политик
Из тестового пода test-pod выполнены запросы ко всем трём сервисам. Все они доступны, что соответствует ожиданию (по умолчанию трафик разрешён).

kubectl exec -n app test-pod -- curl http://frontend-service
Вывод:(before-frontend.txt)

kubectl exec -n app test-pod -- curl http://backend-service
Вывод: (файл before-backend.txt)


kubectl exec -n app test-pod -- curl http://cache-service
Вывод: (файл before-cache.txt)

рис.1_03, рис.1_035

1.3. Создание сетевых политик
Применены три политики, реализующие требуемую цепочку доступа:

frontend-policy: разрешает исходящий трафик к backend (порт 8080), запрещает входящий трафик.

backend-policy: разрешает входящий трафик от frontend и исходящий к cache.

cache-policy: разрешает только входящий трафик от backend, исходящий запрещён (кроме DNS).

Манифесты:

policy-frontend.yaml

policy-backend.yaml

policy-cache.yaml

kubectl apply -f policy-frontend.yaml
kubectl apply -f policy-backend.yaml
kubectl apply -f policy-cache.yaml

1.4. Проверка доступности после применения политик
Разрешённые соединения:

Из пода frontend в backend:

kubectl exec -n app frontend-xxxxx -- curl http://backend-service
Вывод: (файл after-frontend-to-backend.txt)

Из пода backend в cache:

kubectl exec -n app backend-xxxxx -- curl http://cache-service
Вывод: (файл after-backend-to-cache.txt)

Запрещённые соединения:

Из пода cache в frontend:

kubectl exec -n app cache-xxxxx -- curl http://frontend-service
Результат: таймаут или отказ (файл after-cache-to-frontend.txt – пусто или сообщение об ошибке)

Из тестового пода в любой сервис:

kubectl exec -n app test-pod -- curl http://frontend-service
kubectl exec -n app test-pod -- curl http://backend-service
kubectl exec -n app test-pod -- curl http://cache-service
Все три команды должны завершиться ошибкой (файлы after-test-to-frontend.txt, after-test-to-backend.txt, after-test-to-cache.txt содержат сообщения об ошибках).

рис.1_05, рис.1_06,рис. 1_07

1.5. Описание политик

kubectl describe networkpolicy -n app
Вывод сохранён в network-policies-describe.txt

Выводы
В ходе работы были успешно созданы и применены сетевые политики, ограничивающие трафик между микросервисами в соответствии с заданной цепочкой. Продемонстрировано, что до применения политик весь трафик разрешён, а после – разрешены только необходимые соединения, а все остальные заблокированы. Это подтверждает эффективность использования Network Policies для обеспечения сетевой безопасности в Kubernetes.
