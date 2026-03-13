Домашнее задание: Настройка приложений и управление доступом в Kubernetes

Цель работы
Освоить использование ConfigMaps для конфигурации приложений, Secrets для хранения TLS-сертификатов и механизмы RBAC для разграничения доступа пользователей.

Задание 1. Работа с ConfigMaps
1.1. Создание ConfigMap
Создан ConfigMap web-content с содержимым файла index.html.

Манифест: configmap-web.yaml

1.2. Deployment с nginx и multitool
Создан Deployment web-app, включающий два контейнера: nginx (порт 80) и multitool (порт 8080). ConfigMap смонтирован в nginx по пути /usr/share/nginx/html/index.html.

Манифест: deployment.yaml

1.3. Проверка доступности
Выполнен проброс порта и запрос curl:

kubectl port-forward pod/web-app-xxx-xxx 8081:80 &
curl http://localhost:8081
Результат:

html
<!DOCTYPE html>
<html>
<head>
  <title>ConfigMap Demo</title>
</head>
<body>
  <h1>Hello from Kubernetes ConfigMap!</h1>
  <p>Эта страница загружена из ConfigMap.</p>
</body>
</html>
Файл с результатом: configmap-result.txt
рис.1_001

Задание 2. Настройка HTTPS с секретами
2.1. Генерация сертификата и создание Secret
Сгенерирован самоподписанный сертификат для домена myapp.example.com:

openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=myapp.example.com"
Создан Secret tls-secret типа kubernetes.io/tls:

kubectl create secret tls tls-secret --cert=tls.crt --key=tls.key
Альтернативно можно использовать манифест.

Манифест: secret-tls.yaml

2.2. Создание Ingress с TLS
Создан Ingress web-app-ingress, использующий TLS-секрет для домена myapp.example.com.

Манифест: ingress-tls.yaml

2.3. Проверка HTTPS-доступа
В файл /etc/hosts добавлена запись, связывающая внешний IP виртуальной машины с доменом:

62.84.112.77   myapp.example.com
Выполнен запрос с игнорированием проверки сертификата (самоподписанный):

curl -k https://myapp.example.com
Результат: тот же HTML, что и в задании 1.
Файл с результатом: https-result.txt
рис.2_002

Задание 3. Настройка RBAC
3.1. Включение RBAC (в MicroK8s включено по умолчанию)

microk8s enable rbac   # если не включено

3.2. Создание сертификата пользователя developer
Созданы ключ и запрос на сертификат (CSR):

openssl genrsa -out developer.key 2048
openssl req -new -key developer.key -out developer.csr -subj "/CN=developer"
Сертификат подписан с использованием CA кластера MicroK8s (требуются права root для доступа к ca.key):


sudo openssl x509 -req -in developer.csr -CA /var/snap/microk8s/current/certs/ca.crt -CAkey /var/snap/microk8s/current/certs/ca.key -CAcreateserial -out developer.crt -days 365
sudo chown $USER:$USER developer.crt
3.3. Настройка контекста и привязка ролей
Создан пользователь в kubeconfig с абсолютными путями к сертификатам:

kubectl config set-credentials developer \
  --client-certificate=$HOME/DZ_8/developer.crt \
  --client-key=$HOME/DZ_8/developer.key
Создан контекст для пользователя:

kubectl config set-context developer-context \
  --cluster=microk8s-cluster --user=developer
Создание Role и RoleBinding (вместо ClusterRoleBinding, чтобы ограничить доступ только к namespace default):

Role pod-reader (разрешает get, list, watch для pods и pods/log):

yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
RoleBinding developer-pod-reader:

yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-pod-reader
  namespace: default
subjects:
- kind: User
  name: developer
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
Применение манифестов:

kubectl apply -f role-pod-reader.yaml
kubectl apply -f rolebinding-developer.yaml

3.4. Проверка прав доступа
Переключение на контекст разработчика:

kubectl config use-context developer-context
Проверка чтения подов:

kubectl get pods
Вывод (успешно, список подов сохранён в rbac-get-pods.txt):

NAME                                         READY   STATUS    RESTARTS   AGE
web-app-7bb6cf566b-6fccg                     2/2     Running   0          44m
...
Проверка чтения логов:

kubectl logs web-app-7bb6cf566b-6fccg > rbac-logs.txt
Логи успешно сохранены.

Проверка попытки удаления пода:


kubectl delete pod web-app-7bb6cf566b-6fccg 2>&1 | tee rbac-delete-error.txt
Ожидаемая ошибка:

Error from server (Forbidden): pods "web-app-7bb6cf566b-6fccg" is forbidden: User "developer" cannot delete resource "pods" in API group "" in the namespace "default"
Дополнительно: попытка использовать имперсонацию (--as=developer) из контекста администратора приводит к ошибке, так как пользователь developer не имеет права на имперсонацию:

kubectl get pods --as=developer
Ошибка:

Error from server (Forbidden): users "developer" is forbidden: User "developer" cannot impersonate resource "users" in API group "" at the cluster scope
Это подтверждает, что RBAC настроен корректно.

Файлы с результатами:

rbac-get-pods.txt — вывод kubectl get pods

rbac-logs.txt — логи пода

rbac-delete-error.txt — ошибка при удалении

рис.3_004

Выводы
В ходе работы были успешно решены следующие задачи:

Создание ConfigMap и его использование для конфигурации веб-страницы в nginx.

Генерация TLS-сертификата, создание Secret и настройка Ingress для HTTPS-доступа.

Включение RBAC, создание пользовательского сертификата, подписанного CA кластера, настройка Role и RoleBinding для ограничения прав доступа.

Проверка корректности настроек через переключение контекста и выполнение команд kubectl, демонстрирующих разрешённые (чтение) и запрещённые (удаление) действия.

Все манифесты и файлы с результатами находятся в папке DZ_8.
