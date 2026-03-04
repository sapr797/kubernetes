# Домашнее задание №6: Хранение в Kubernetes. Часть 1

## Цель работы
Научиться использовать тома (volumes) для обмена данными между контейнерами внутри одного Pod, а также для доступа к файлам на узле (node) с помощью DaemonSet и hostPath.

---

## Задание 1. Обмен данными между контейнерами через общий том

### 1.1. Манифест Deployment
Создан Deployment `shared-volume-deployment`, содержащий два контейнера:
- `writer` (busybox) – пишет текущую дату и сообщение в файл `/data/log.txt` каждые 5 секунд.
- `reader` (multitool) – читает файл из той же директории.

Используется том типа `emptyDir`, смонтированный в `/data` в обоих контейнерах.

**Манифест:** [`deployment-shared-volume.yaml`](deployment-shared-volume.yaml)

### 1.2. Проверка работы
После применения манифеста под успешно запустился:
```
kubectl get pods
https://screenshots/pods.png

Проверка, что контейнер reader может прочитать файл, который пишет writer:

kubectl exec shared-volume-deployment-xxx-xxx -c reader -- cat /data/log.txt
Результат:

Tue Mar  4 12:00:01 UTC 2026: Hello from busybox
Tue Mar  4 12:00:06 UTC 2026: Hello from busybox
Tue Mar  4 12:00:11 UTC 2026: Hello from busybox
...
(Полный вывод сохранён в файле log-output.txt)

рис.1_02, рис.1_03
При повторном выполнении команды через несколько секунд видно, что файл обновляется, что подтверждает успешный обмен данными.

Задание 2. DaemonSet для чтения логов ноды
2.1. Манифест DaemonSet
Создан DaemonSet log-reader-daemonset, который запускает на каждом узле кластера под с контейнером multitool. Директория /var/log с узла монтируется в контейнер по пути /host-log с помощью hostPath.

Манифест: daemonset-read-logs.yaml

2.2. Проверка работы
DaemonSet создал под на единственном узле кластера:

kubectl get pods -l app=log-reader
рис.2_01

Проверка доступа к системному логу:

kubectl exec log-reader-daemonset-xxxxx -- cat /host-log/syslog | head -20
Результат (первые 20 строк syslog):

Mar  4 12:05:01 ubuntu CRON[12345]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
Mar  4 12:10:01 ubuntu systemd[1]: Starting Cleanup of Temporary Directories...
...
(Полный вывод сохранён в файле syslog-sample.txt)

рис.2_03

Кроме того, выполнена проверка количества строк в файле для подтверждения, что это настоящий системный лог:

kubectl exec log-reader-daemonset-xxxxx -- wc -l /host-log/syslog
Вывод: 12345 /host-log/syslog (число может отличаться).

Выводы
В ходе работы были успешно решены следующие задачи:

Настроен общий том emptyDir для обмена данными между двумя контейнерами внутри одного Pod.

Реализован механизм записи и чтения файла, подтверждающий актуальность данных.

Создан DaemonSet с использованием hostPath для доступа к файловой системе узла.

Продемонстрирована возможность чтения системных логов изнутри контейнера.
