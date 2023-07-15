# d1manxoloda_platform
d1manxoloda Platform repository

## Домашнее задание №1
- [x] **Основное ДЗ:**
Разберитесь почему все pod в namespace **kube-system** восстановились после удаления
#### Ответ:
Причина почему все pod-ы в namespace **kube-system** восстановились, заключается в том что все pod-ы за исключением coredns
являются [static pod-ами] и управляются напрямую.
Pod **coredns** описан в деплойменте неймспейса kube-system.

Dockerfile
- Создан докерфайл, описывающий образ веб-сервера, прослушивающего порт 8000 и позволяющего просматривать файлы из директории app.
- Собран образ d1manxoloda/mynginx:otus, на основе alpine и запушен в Dockerhub.

Манифест pod
- Создан манифест web-pod.yaml
- В манифест добавлен initContainer, генерирующий index.html.
- Добален Volumes для доступности генерируемых в initContainers файлов в контейнере web.
- Под запущен, выполнена переадресация портов пода, веб-сервер работает, страница открывается.
  ```
  kubectl port-forward --address 0.0.0.0 pod/web 8000:8000
  ```

Hipster ShopHipster Shop
- Склонирован [репозиторий](https://github.com/GoogleCloudPlatform/microservices-demo).
- На основе имеющегося в репозитории dockerfile собран и запушен в dockerhub образ d1manxoloda/hipster-shop:otus
- Сгенерирован манифест frontend-pod.yaml.
- Выполнен запуск пода с манифкста frontend-pod.yaml, под находится в статусе Error.
  ```
  kubectl run frontend --image d1manxoloda/hipster-shop:otus --restart=Never --dry-run -o yaml > frontend-pod.yaml
  ```
Hipster Shop | Задание со *
- Под frontend находится в статусе Error, из-за отсутствия env:
  ```
  panic: environment variable "PRODUCT_CATALOG_SERVICE_ADDR" not set
  panic: environment variable "CURRENCY_SERVICE_ADDR" not set
  panic: environment variable "CART_SERVICE_ADDR" not set
  panic: environment variable "RECOMMENDATION_SERVICE_ADDR" not set
  panic: environment variable "CHECKOUT_SERVICE_ADDR" not set
  panic: environment variable "SHIPPING_SERVICE_ADDR" not set
  panic: environment variable "AD_SERVICE_ADDR" not set
  ```
- Создан новый манифест frontend-pod-healthy.yaml, с указанием env.
- Запущенный с манифеста frontend-pod-healthy.yaml, под frontend находится в статусе Running.

## Домашнее задание №2

## В процессе сделано:
  - Пункт 1
1. Был создан первый манифест ReplicaSet **frontend-replicaset.yaml** из методички HW2
  
   1. Были исправлены ошибки в манифесте. Добавлена секция selector. 
   2. В манифест добавлены переменные (env), которые необходимы подам для работоспособности.
   3. Через команду  `kubectl apply -f frontend-replicaset.yaml` запустили репликасет. Убедились что поднялась одна реплика пода **frontend** командой `kubectl get rs frontend`
      ```
      kubectl get rs frontend
      NAME       DESIRED   CURRENT   READY   AGE
      frontend   1         1         1       46s
      ```
   4. Изменили количество реплик с 1 на 3 через команду `kubectl scale replicaset frontend --replicas=3`. Убедились, что количество реплик пода **frontend** изменилось на 3 командой `kubectl get rs frontend` 
      ```
      kubectl get rs frontend
      NAME       DESIRED   CURRENT   READY   AGE
      frontend   3         3         3       46s
      ```
   5. С помощью команды  `kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w` убедились, что репликасет отслеживает изменения состояний подов и поднимает новые реплики. В нашем случае они были специально удалены.
      ```
      kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w
      NAME             READY   STATUS              RESTARTS   AGE
      frontend-g4df2   0/1     ContainerCreating   0          3s
      frontend-92kls   0/1     ContainerCreating   0          3s
      frontend-2dsp4   0/1     ContainerCreating   0          3s
      frontend-2dsp4   1/1     Running             0          4s
      frontend-g4df2   1/1     Running             0          4s
      frontend-92kls   1/1     Running             0          5s
      ```
   6. Изменен манифест, чтобы сразу поднималось 3 реплики пода.
   7. Изменена версия образа с которого репликасет поднимает под. После применения манифеста командой `kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'` убеждаемся что образ с которого будут создаваться реплики подов изменился на указанную нами в манифесте версию
   8. Добавлены deployment манифесты для paymentservice.
   9. Добавлен Probes для frontend


- Пункт 2
1. Добавлены deployment манифесты для paymentservice с двумя стратегиями
   1. Аналог blue-green:
      ```
      spec:
         strategy:
           type: RollingUpdate
           rollingUpdate:
             maxSurge: 0
             maxUnavailable: 1
      ```
   2. Reverse Rolling Update:
      ```
      spec:
         strategy:
           type: RollingUpdate
           rollingUpdate:
             maxSurge: 3
             maxUnavailable: 0
      ```
2 Добавлен node-exporter-daemonset.yaml.
     ```
      spec:
        tolerations:
          - key: node-role.kubernetes.io/master
            operator: Exists
            effect: NoSchedule
     ```
