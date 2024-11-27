**Задание 1**

Создать Deployment приложения, состоящего из двух контейнеров и обменивающихся данными.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.

2. Сделать так, чтобы busybox писал каждые пять секунд в некий файл в общей директории.

3. Обеспечить возможность чтения файла контейнером multitool.

4. Продемонстрировать, что multitool может читать файл, который периодоически обновляется.

5. Предоставить манифесты Deployment в решении, а также скриншоты или вывод команды из п. 4.



**Решение 1**

Создадим новый namespace 

![Image alt](https://github.com/mezhibo/kubernetes6/blob/4a1481ef20491e95a85f59381e6e567d2de93c7f/IMG/1.jpg)


Далее создадим деплоймент со следующим содержимым

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netology-app-multitool-busybox
  namespace: kube6
spec:
  replicas: 1
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ['sh', '-c', 'while true; do echo "current date = $(date)" >> /busybox_dir/date.log; sleep 5; done']
        volumeMounts:
          - mountPath: "/busybox_dir"
            name: deployment-volume
      - name: multitool
        image: wbitt/network-multitool
        volumeMounts:
          - name: deployment-volume
            mountPath: "/multitool_dir"
      volumes:
        - name: deployment-volume
          emptyDir: {}
```


Запускаем манифест, заходим в под и проверяем доступность файлов


![Image alt](https://github.com/mezhibo/kubernetes6/blob/4a1481ef20491e95a85f59381e6e567d2de93c7f/IMG/2.jpg)


Переходим в нужную нам папку и видим файл, созданный в контейнере Busybox доступен для чтения контейнером multitool в котром хранятся логи


![Image alt](https://github.com/mezhibo/kubernetes6/blob/4a1481ef20491e95a85f59381e6e567d2de93c7f/IMG/3.jpg)




**Задание 2**

Создать DaemonSet приложения, которое может прочитать логи ноды.

1. Создать DaemonSet приложения, состоящего из multitool.

2. Обеспечить возможность чтения файла /var/log/syslog кластера MicroK8S.

3. Продемонстрировать возможность чтения файла изнутри пода.

4. Предоставить манифесты Deployment, а также скриншоты или вывод команды из п. 2.


**Решение 2**

Создадим DaemonSet со следующим содержимым, и прекрипим его в прошлый namespace

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: multitool-logs
  namespace: kube6
spec:
  selector:
    matchLabels:
      app: mt-logs
  template:
    metadata:
      labels:
        app: mt-logs
    spec:
      containers:
        - name: multitool
          image: wbitt/network-multitool
          volumeMounts:
            - name: log-volume
              mountPath: /log_data
      volumes:
        - name: log-volume
          hostPath:
            path: /var/log
```


Запускаем манифест и проверяем доступность указанного файла:


![Image alt](https://github.com/mezhibo/kubernetes6/blob/4a1481ef20491e95a85f59381e6e567d2de93c7f/IMG/4.jpg)


![Image alt](https://github.com/mezhibo/kubernetes6/blob/4a1481ef20491e95a85f59381e6e567d2de93c7f/IMG/5.jpg)



И теперь просмотрим содержимое файла логов внутри пода. Как видим, все супер


![Image alt](https://github.com/mezhibo/kubernetes6/blob/4a1481ef20491e95a85f59381e6e567d2de93c7f/IMG/6.jpg)




