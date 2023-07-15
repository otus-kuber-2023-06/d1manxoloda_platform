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
Hipster Shop | Задание со ⭐
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
