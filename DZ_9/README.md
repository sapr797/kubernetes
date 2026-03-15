# Домашнее задание №9: Управление приложениями с помощью Helm

## Цель работы
Освоить создание Helm-чартов, параметризацию конфигурации для разных окружений и развертывание нескольких версий приложения в разных namespace.

---

## Задание 1. Подготовка Helm-чарта для приложения

### 1.1. Создание чарта
Создан Helm-чарт `multi-app` со следующей структурой:
multi-app/
├── Chart.yaml # метаданные чарта
├── values.yaml # значения по умолчанию
└── templates/ # шаблоны манифестов
├── frontend-deployment.yaml
├── frontend-service.yaml
├── backend-deployment.yaml
└── backend-service.yaml


**Chart.yaml:**
```
apiVersion: v2
name: multi-app
description: A Helm chart for multi-component application
type: application
version: 0.1.0
appVersion: "1.0.0"

рис.1_00

1.2. Параметризация образа
В values.yaml определены параметры для обоих компонентов, включая теги образов:

yaml
frontend:
  image:
    repository: nginx
    tag: latest
backend:
  image:
    repository: wbitt/network-multitool
    tag: latest

рис.1_01

1.3. Установка первой версии

kubectl create namespace app1
helm install my-app-v1 ./multi-app --namespace app1

1.4. Обновление версии
Создан файл values-v2.yaml с новыми тегами:

frontend:
  image:
    tag: alpine
backend:
  image:
    tag: extra
Выполнено обновление:

helm upgrade my-app-v1 ./multi-app --namespace app1 --values values-v2.yaml

Задание 2. Запуск нескольких версий в разных namespace
2.1. Вторая версия в том же namespace (app1)

helm install my-app-v1-second ./multi-app --namespace app1 \
  --set frontend.image.tag=perl \
  --set backend.image.tag=alpine-extra

2.2. Третья версия в другом namespace (app2)
kubectl create namespace app2
helm install my-app-v2 ./multi-app --namespace app2 \
  --set frontend.image.tag=stable \
  --set backend.image.tag=fedora

рис.2_02

2.3. Результаты
Все установленные релизы:

helm list --all-namespaces
Вывод сохранён в helm-releases.txt.

Поды в namespace app1 (два релиза):

kubectl get pods -n app1
Вывод сохранён в pods-app1.txt.

Поды в namespace app2 (один релиз):

kubectl get pods -n app2
Вывод сохранён в pods-app2.txt.

рис.2_03

2.4. Проверка доступности приложений
Проверим доступ к frontend из app1:

kubectl port-forward -n app1 pod/my-app-v1-frontend-xxx 8081:80 &
curl http://localhost:8081
Аналогично для других релизов — все приложения работают независимо.

Выводы
В ходе работы были успешно решены следующие задачи:
Создан параметризованный Helm-чарт для многокомпонентного приложения.
Реализована возможность изменения версий образов через values-файлы.
Выполнено развертывание трёх экземпляров приложения:
первая версия в app1,
вторая версия в том же app1 (другой релиз),
третья версия в app2.
Продемонстрирована независимость релизов и их изоляция по namespace.

Все манифесты чарта и подтверждающие файлы находятся в папке DZ_9.



