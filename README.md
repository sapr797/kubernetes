# Лабораторные работы по Kubernetes

**Выполнил:** [Ваше имя]  
**Репозиторий:** [github.com/sapr797/kubernetes](https://github.com/sapr797/kubernetes)

## Содержание
1. [Задание 1. Установка MicroK8s и настройка доступа](#задание-1-установка-microk8s-и-настройка-доступа)
2. [Задание 2. Базовые объекты K8s: Pod и Service](#задание-2-базовые-объекты-k8s-pod-и-service)
3. [Выводы](#выводы)

---

## Задание 1. Установка MicroK8s и настройка доступа

### Цель
Установить MicroK8s на виртуальную машину, включить панель управления (Dashboard), сгенерировать сертификат для подключения по внешнему IP, настроить локальный `kubectl` для доступа к кластеру.

### Выполнение

#### 1. Установка MicroK8s на ВМ (Ubuntu)
```
sudo snap install microk8s --classic
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
newgrp microk8s
microk8s status --wait-ready

2. Включение Dashboard
microk8s enable dashboard
Получение токена для входа:

microk8s kubectl -n kube-system create token admin-user
Токен сохранён в файле DZ_1/Vm/dashboard-token.txt.

3. Генерация сертификата для внешнего IP
На ВМ отредактирован файл /var/snap/microk8s/current/certs/csr.conf.template – добавлен внешний IP 93.77.189.168 в секцию [ alt_names ]:

IP.5 = 93.77.189.168
Обновлён сертификат API-сервера:

sudo microk8s refresh-certs --cert server.crt
MicroK8s перезапущен:

microk8s stop && microk8s start
4. Настройка локального kubectl (на WSL)
Скопирован конфиг с ВМ и изменён server на внешний IP:

scp alexlinux@93.77.189.168:~/.kube/config ~/.kube/config
sed -i 's/127.0.0.1/93.77.189.168/' ~/.kube/config
Проверка подключения:

kubectl get nodes
Вывод:

NAME         STATUS   ROLES    AGE   VERSION
kubernetes   Ready    <none>   44h   v1.33.7
5. Результаты
В папке DZ_1 репозитория сохранены:

Vm/ – файлы, полученные с виртуальной машины:

kubeconfig – конфигурация для подключения к кластеру.

csr.conf.template – шаблон CSR с добавленным внешним IP.

microk8s-certs/ – публичные сертификаты MicroK8s.

nodes.txt, pods.txt – выводы команд.

dashboard-token.txt – токен для входа в Dashboard.

Wsl/ – файлы, созданные на локальной WSL:

dashboard.png – скриншот открытой панели Dashboard.

wsl_history.txt – история команд в WSL.

kubeconfig-wsl – локальный конфиг kubectl.

Задание 2. Базовые объекты K8s: Pod и Service
Цель
Развернуть Pod с приложением echoserver и подключиться к нему через kubectl port-forward. Затем создать Service и подключиться к нему аналогично.

Выполнение
1. Создание Pod hello-world
Манифест pod-hello-world.yaml:

yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
  labels:
    app: hello-world
spec:
  containers:
  - name: echoserver
    image: gcr.io/kubernetes-e2e-test-images/echoserver:2.2
    ports:
    - containerPort: 8080
Применение:

kubectl apply -f pod-hello-world.yaml
Проверка порта внутри пода:

kubectl exec hello-world -- netstat -tulpn | grep LISTEN
Вывод:

tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      9/nginx: master pro
Порт приложения – 8080.

Подключение через port-forward:

kubectl port-forward pod/hello-world 8080:8080
В другом окне:

curl http://localhost:8080
Ответ (сокращён):

Hostname: hello-world
Request Information:
        client_address=127.0.0.1
        method=GET
        ...
2. Создание Pod netology-web и Service netology-svc
Манифест pod-netology-web.yaml:

yaml
apiVersion: v1
kind: Pod
metadata:
  name: netology-web
  labels:
    app: netology
spec:
  containers:
  - name: echoserver
    image: gcr.io/kubernetes-e2e-test-images/echoserver:2.2
    ports:
    - containerPort: 8080
Манифест service-netology-svc.yaml:

yaml
apiVersion: v1
kind: Service
metadata:
  name: netology-svc
spec:
  selector:
    app: netology
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
Применение:

kubectl apply -f pod-netology-web.yaml
kubectl apply -f service-netology-svc.yaml
Проверка endpoints:

kubectl get endpoints netology-svc
Вывод:

NAME           ENDPOINTS           AGE
netology-svc   10.1.192.100:8080   4s
Подключение через port-forward к сервису:

kubectl port-forward service/netology-svc 8081:80
В другом окне:

curl http://localhost:8081
Ответ (сокращён):


Hostname: netology-web
Request Information:
        client_address=127.0.0.1
        method=GET
        ...
3. Сохранение результатов
Все манифесты и выводы команд сохранены в папке DZ2:

pod-hello-world.yaml

pod-netology-web.yaml

service-netology-svc.yaml

pods.txt – результат kubectl get pods

services.txt – результат kubectl get svc

endpoints.txt – результат kubectl get endpoints

curl-pod.txt – ответ curl на порт 8080 (прямо к поду)

curl-svc.txt – ответ curl на порт 8081 (через сервис)

Выводы
В ходе выполнения лабораторных работ были достигнуты следующие результаты:

Установлен и настроен MicroK8s с включённым Dashboard.

Сгенерирован сертификат для доступа по внешнему IP, что позволило подключаться к кластеру удалённо.

Освоена работа с kubectl, созданы манифесты Pod и Service.

Проверена работа port-forward для прямого доступа к поду и через сервис.

Все конфигурационные файлы и результаты верификации загружены в Git-репозиторий.

Таким образом, получены практические навыки развёртывания базовых объектов Kubernetes и управления ими.
