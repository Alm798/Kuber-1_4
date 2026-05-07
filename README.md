## Домашнее задание к занятию «Сетевое взаимодействие в Kubernetes»
## Михеев Алексей


### Чеклист готовности

- Установлен Kubernetes (MicroK8S, Minikube или другой).

- Установлен kubectl.

- Редактор для YAML-файлов (VS Code, Vim и др.).


### Инструменты, которые пригодятся для выполнения задания

- Инструкция по установке MicroK8S.

- Инструкция по установке Minikube.

- Инструкцияпо установке kubectl.

- Инструкция по установке VS Code


### Дополнительные материалы, которые пригодятся для выполнения задания

- Описание Deployment и примеры манифестов.

- Описание Описание Service.

- Описание Ingress.

- Описание Multitool.



### Задание 1: Настройка Service (ClusterIP и NodePort)

#### Задача

Развернуть приложение из двух контейнеров (nginx и multitool) и обеспечить доступ к ним:

- Внутри кластера через ClusterIP.

- Снаружи через NodePort.

#### Шаги выполнения

1. Создать Deployment с двумя контейнерами:
 
 - nginx (порт 80).
 
 - multitool (порт 8080).
 
 - Количество реплик: 3.

2. Создать Service типа ClusterIP, который:

 - Открывает nginx на порту 9001.

 - Открывает multitool на порту 9002.

3. Проверить доступность изнутри кластера:

``` 
 kubectl run test-pod --image=wbitt/network-multitool --rm -it -- sh
 curl <service-name>:9001 # Проверить nginx
 curl <service-name>:9002 # Проверить multitool
```

4. Создать Service типа NodePort для доступа к nginx снаружи.

5. Проверить доступ с локального компьютера:

 ```
 curl <node-ip>:<node-port> или через браузер.
```
#### Что сдать на проверку

- Манифесты:
```
deployment-multi-container.yaml
service-clusterip.yaml
service-nodeport.yaml
```
- Скриншоты проверки доступа (curl или браузер).

- - - - -

### Решение:

![1](https://github.com/Ivan-Shkutov/kuber_1.4/blob/main/jpg/1.png)

![2](https://github.com/Ivan-Shkutov/kuber_1.4/blob/main/jpg/2.png)

![3](https://github.com/Ivan-Shkutov/kuber_1.4/blob/main/jpg/3.2.png)

![4](https://github.com/Ivan-Shkutov/kuber_1.4/blob/main/jpg/3.3.png)

![5](https://github.com/Ivan-Shkutov/kuber_1.4/blob/main/jpg/3.png)


```
1. Проверяем статус Minikube:
minikube status

2. Запускаем Minikube (если не запущен):
minikube start

Minikube создаёт локальную виртуальную машину с Kubernetes.
После запуска API-сервер будет доступен, kubectl автоматически настроится.

3. Проверяем, что kubectl видит ноды:
kubectl get nodes

4. Применяем Deployment:
kubectl apply -f deployment-multi-container.yaml

5. Проверяем поды:
kubectl get pods

6. Применяем Service:
kubectl apply -f service-clusterip.yaml

7. Проверяем сервис:
kubectl get svc

8. Создаём временный тестовый pod:
kubectl run test-pod --image=wbitt/network-multitool --rm -it -- sh

9. Проверяем доступ к сервису:
curl multi-container-clusterip:9001
curl multi-container-clusterip:9002

10. Применяем Service:
kubectl apply -f service-nodeport.yaml

11. Проверяем сервис:
kubectl get svc

12. Проверяем curl с локального компьютера:
curl http://192.168.49.2:30080
```



- - - - -

### Задание 2: Настройка Ingress

#### Задача

Развернуть два приложения (frontend и backend) и обеспечить доступ к ним через Ingress по разным путям.

#### Шаги выполнения

1. Развернуть два Deployment:
```
  frontend (образ nginx).
  backend (образ wbitt/network-multitool).
```
2. Создать Service для каждого приложения.

3. Включить Ingress-контроллер:
``` 
 microk8s enable ingress
```
4. Создать Ingress, который:

 - Открывает frontend по пути /.
 - Открывает backend по пути /api.

5. Проверить доступность:
 ```
 curl <host>/
 curl <host>/api
или через браузер.
```
#### Что сдать на проверку

Манифесты:
```
deployment-frontend.yaml
deployment-backend.yaml
service-frontend.yaml
service-backend.yaml
ingress.yaml
```
Скриншоты проверки доступа (curl или браузер).


#### Шаблоны манифестов с учебными комментариями

1. Deployment (nginx + multitool)

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: # ПРИМЕР: "multi-container-app"
spec:
  replicas: # ЗАДАНИЕ: Укажите количество реплик
  selector:
    matchLabels:
      app: # ДОПОЛНИТЕ: Метка для селектора
  template:
    metadata:
      labels:
        app: # ПОВТОРИТЕ метку из selector.matchLabels
    spec:
      containers:
 - name: # ЗАДАНИЕ: Название первого контейнера
        image: nginx
        ports:
 - containerPort: 80
 - name: multitool
        image: wbitt/network-multitool
        ports:
 - containerPort: 8080
        env:
 - name: HTTP_PORT
          value: "8080" # КЛЮЧЕВОЙ МОМЕНТ: Порт должен совпадать с containerPort
```


2. Ingress (для frontend и backend)

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: # ЗАДАНИЕ: Придумайте имя, допустим example-ingress
  annotations:  # ВАЖНО: Эта аннотация нужна для rewrite правил
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
 - http:
      paths:
 - path: /
        pathType: Prefix
        backend:
          service:
            name: # УКАЖИТЕ: Имя frontend Service
            port:
              number: 80
 - path: /api # КЛЮЧЕВОЙ ПУТЬ: API endpoint
        pathType: Prefix
        backend:
          service:
            name: # УКАЖИТЕ: Имя backend Service
            port:
              number: 80
```


- - - - -

### Решение:


![6](https://github.com/Ivan-Shkutov/kuber_1.4/blob/main/jpg/4.1.png)

![7](https://github.com/Ivan-Shkutov/kuber_1.4/blob/main/jpg/4.2.png)

![8](https://github.com/Ivan-Shkutov/kuber_1.4/blob/main/jpg/4.3.png)


```
1. Проверяем статус Minikube:
minikube status

2. Запускаем Minikube (если не запущен):
minikube start

3. Включение Ingress (в Minikube Ingress не работает без включения addons ingress):
minikube addons enable ingress

4. Проверяем, что pod запущен:
kubectl get pods -n kube-system | grep ingress

5. Применение всех манифестов:
kubectl apply -f deployment-frontend.yaml
kubectl apply -f deployment-backend.yaml
kubectl apply -f service-frontend.yaml
kubectl apply -f service-backend.yaml
kubectl apply -f ingress.yaml

6. Проверяем:
kubectl get pods
kubectl get svc
kubectl get ingress

7. Проверка доступа через Ingress:
minikube ip

8. Проверяем frontend:
curl http://192.168.49.2/

9. Проверяем backend:
curl http://192.168.49.2/api

10. Проверка через браузер:
http://192.168.49.2/       # frontend
http://192.168.49.2/api    # backend
```

- - - - -




