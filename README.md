# Домашнее задание к занятию «Сетевое взаимодействие в Kubernetes» - Морозов Александр

### Примерное время выполнения задания

120 минут

### Цель задания

Научиться настраивать доступ к приложениям в Kubernetes:
- Внутри кластера через **Service** (ClusterIP, NodePort).
- Снаружи кластера через **Ingress**.

Это задание поможет вам освоить базовые принципы сетевого взаимодействия в Kubernetes — ключевого навыка для работы с кластерами.
На практике Service и Ingress используются для доступа к приложениям, балансировки нагрузки и маршрутизации трафика. Понимание этих механизмов поможет вам упростить управление сервисами в рабочих окружениях и снизит риски ошибок при развёртывании.

------

## **Подготовка**
### **Чеклист готовности**
- Установлен Kubernetes (MicroK8S, Minikube или другой).
- Установлен `kubectl`.
- Редактор для YAML-файлов (VS Code, Vim и др.).

------

### Инструменты, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S.
2. [Инструкция](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download) по установке Minikube. 
3. [Инструкция](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)по установке kubectl.
4. [Инструкция](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools) по установке VS Code

### Дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Описание Service.
3. [Описание](https://kubernetes.io/docs/concepts/services-networking/ingress/) Ingress.
4. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

## **Задание 1: Настройка Service (ClusterIP и NodePort)**
### **Задача**
Развернуть приложение из двух контейнеров (`nginx` и `multitool`) и обеспечить доступ к ним:
- Внутри кластера через **ClusterIP**.
- Снаружи через **NodePort**.

### **Шаги выполнения**
1. **Создать Deployment** с двумя контейнерами:
   - `nginx` (порт `80`).
   - `multitool` (порт `8080`).
   - Количество реплик: `3`.
2. **Создать Service типа ClusterIP**, который:
   - Открывает `nginx` на порту `9001`.
   - Открывает `multitool` на порту `9002`.
3. **Проверить доступность** изнутри кластера:
```bash
 kubectl run test-pod --image=wbitt/network-multitool --rm -it -- sh
 curl <service-name>:9001 # Проверить nginx
 curl <service-name>:9002 # Проверить multitool
```
4. **Создать Service типа NodePort** для доступа к `nginx` снаружи.
5. **Проверить доступ** с локального компьютера:
```bash
 curl <node-ip>:<node-port>
   ```
 или через браузер.

### **Что сдать на проверку**
- Манифесты:
  - `deployment-multi-container.yaml`
  - `service-clusterip.yaml`
  - `service-nodeport.yaml`
- Скриншоты проверки доступа (`curl` или браузер).


Ответ:
1. Манифест deployment с двумя контейнерами Nginx и MultiTool [deploy.yml](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/k8s/deploy.yml)
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep-multi-container
  labels:
    app: multi_web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: multi_web
  template:
    metadata:
      labels:
        app: multi_web
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: network-multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
        env:
          - name: HTTP_PORT
            value: "8080"
```

2. Манифест service типа ClusterIP [service.yml](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/k8s/service.yml)
```
apiVersion: v1
kind: Service
metadata:
  name: svc-multi-container
spec:
  ports:
  - name: nginx-svc
    protocol: TCP
    port: 9001
    targetPort: 80
  - name: multitool-svc
    protocol: TCP
    port: 9002
    targetPort: 8080
  selector:
    app: multi_web
  type: ClusterIP
```
![alt text](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/img/1.png)

3. Проверяем доступность изнутри кластера
![alt text](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/img/2.png)

4. Манифест service типа NodePort [service-np.yml](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/k8s/service-np.yml)
```
apiVersion: v1
kind: Service
metadata:
  name: svc-multi-container-np
spec:
  ports:
  - name: nginx-svc
    protocol: TCP
    port: 9001
    targetPort: 80
    nodePort: 30080
  - name: multitool-svc
    protocol: TCP
    port: 9002
    targetPort: 8080
  selector:
    app: multi_web 
  type: NodePort 
```

5. Проверяем доступ** с локального компьютера
![alt text](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/img/3.png)

![alt text](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/img/4.png)

![alt text](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/img/5.png)


---
## **Задание 2: Настройка Ingress**
### **Задача**
Развернуть два приложения (`frontend` и `backend`) и обеспечить доступ к ним через **Ingress** по разным путям.

### **Шаги выполнения**
1. **Развернуть два Deployment**:
   - `frontend` (образ `nginx`).
   - `backend` (образ `wbitt/network-multitool`).
2. **Создать Service** для каждого приложения.
3. **Включить Ingress-контроллер**:
```bash
 microk8s enable ingress
   ```
4. **Создать Ingress**, который:
   - Открывает `frontend` по пути `/`.
   - Открывает `backend` по пути `/api`.
5. **Проверить доступность**:
```bash
 curl <host>/
 curl <host>/api
   ```
 или через браузер.


 Ответ:
 1. Манифест deployment backend [deploy-back.yml](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/k8s/deploy-back.yml)
 ```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep-back
  labels:
    app: back
spec:
  selector:
    matchLabels:
      app: back
  template:
    metadata:
      labels:
        app: back
    spec:
      containers:
      - name: network-multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
        env:
          - name: HTTP_PORT
            value: "8080"
 ```

 Манифест deployment frontend [deploy-front.yml](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/k8s/deploy-front.yml)
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep-front
  labels:
    app: front
spec:
  selector:
    matchLabels:
      app: front
  template:
    metadata:
      labels:
        app: front
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        env:
          - name: HTTP_PORT
            value: "80"
```

2. Манифест service backend [service-back.yml](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/k8s/service-back.yml)
```
apiVersion: v1
kind: Service
metadata:
  name: svc-back
spec:
  ports:
  - name: svc-back
    protocol: TCP
    port: 9002
    targetPort: 8080
  selector:
    app: back 
  type: NodePort 
```

Манифест service frontend [service-front.yml](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/k8s/service-front.yml)
```
apiVersion: v1
kind: Service
metadata:
  name: svc-front
spec:
  ports:
  - name: svc-front
    protocol: TCP
    port: 9001
    targetPort: 80
    nodePort: 30080
  selector:
    app: front 
  type: NodePort
```

3. Запускаем Ingress-контроллер
![alt text](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/img/6.png)

4. Манифест ingress [ingress.yml](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/k8s/ingress.yml)
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-front-back
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
            name: svc-front # УКАЖИТЕ: Имя frontend Service
            port:
              number: 9001
      - path: /api # КЛЮЧЕВОЙ ПУТЬ: API endpoint
        pathType: Prefix
        backend:
          service:
            name: svc-back # УКАЖИТЕ: Имя backend Service
            port:
              number: 9002
```

![alt text](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/img/7.png)

5. Проверяем доступность 
Прописываем в hosts ВМ K8S
```
127.0.0.1 netology-ingress-f-b.ru
```
На локальном АРМ прописываем в hosts внешний IP ВМ K8S для домена netology-ingress-f-b.ru

![alt text](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/img/8.png)

![alt text](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/img/9.png)

![alt text](https://github.com/Mars12121/kuber-homeworks_1.4/blob/main/img/10.png)


### **Что сдать на проверку**
- Манифесты:
  - `deployment-frontend.yaml`
  - `deployment-backend.yaml`
  - `service-frontend.yaml`
  - `service-backend.yaml`
  - `ingress.yaml`
- Скриншоты проверки доступа (`curl` или браузер).

---
## Шаблоны манифестов с учебными комментариями
### **1. Deployment (nginx + multitool)**
```yaml
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
### **2. Ingress (для frontend и backend)**
```yaml
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
---

## **Правила приёма работы**
1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

## **Критерии оценивания задания**
1. Зачёт: Все задачи выполнены, манифесты корректны, есть доказательства работы (скриншоты).
2. Доработка (на доработку задание направляется 1 раз): основные задачи выполнены, при этом есть ошибки в манифестах или отсутствуют проверочные скриншоты.
3. Незачёт: работа выполнена не в полном объёме, есть ошибки в манифестах, отсутствуют проверочные скриншоты. Все попытки доработки израсходованы (на доработку работа направляется 1 раз). Этот вид оценки используется крайне редко.

## **Срок выполнения задания**  
1. 5 дней на выполнение задания.
2. 5 дней на доработку задания (в случае направления задания на доработку).