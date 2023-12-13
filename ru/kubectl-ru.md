### Table of Contents
- **[README](../README.md)**
- **Ansible** (English/[Russian](ansible-ru.md)): Automation of configuration and system management.
- **Elasticsearch** (English/[Russian](elastic-ru.md)): Search and data analysis.
- **Kubernetes** (English/[Russian](kubectl-ru.md)): Container orchestration and cluster management.
- **Regular Expressions** (English/[Russian](regulars-ru.md)): A powerful tool for working with text.
- **Ubuntu** (English/[Russian](ubuntu-ru.md)): Basic settings and recommendations for using Ubuntu.
- **HashiCorp Vault** ([English](../en/vault-en.md)/Russian): Management of secrets and access.

# Терминология и концепции

**Kubernetes (K8s)** - Система управления контейнерами, разработанная для автоматизации развертывания, масштабирования и управления приложениями в контейнерах.

**Под (Pod)** - Самая маленькая управляемая единица в Kubernetes, содержащая один или несколько контейнеров, разделяющих сеть и хранилище.

**Сервис (Service)** - Абстракция, предоставляющая сетевой доступ (ip, port, etc.) к набору подов, обеспечивая балансировку нагрузки и отказоустойчивость.

**Ingress** - Ресурс, который управляет внешним доступом к службам в кластере.

**Репликация (Replication)** - Механизм, который обеспечивает масштабирование приложений в Kubernetes путем создания и управления несколькими идентичными репликами.

**Маштабирование горизонтальное (Horizontal Scaling)** - Процесс автоматического увеличения или уменьшения количества реплик приложения для обеспечения баланса нагрузки и устойчивости.

**Мастер-узел (Master Node)** - Основной компонент Kubernetes, который управляет кластером, принимает решения о развертывании и управлении подами.

**Рабочая нода (Worker Node)** - Узел в кластере Kubernetes, на котором запускаются и работают поды.

**Хранилище данных (Persistent Volume)** - Механизм, который предоставляет постоянное хранилище для приложений в Kubernetes.

**Оператор (Operator)** - Пользовательский контроллер, который расширяет возможности Kubernetes для управления комплексными приложениями.

**Kubelet** - Компонент на рабочей ноде, отвечающий за выполнение действий на этой ноде и управление подами.

**YAML** - Формат файлов для описания ресурсов Kubernetes, используется для конфигурации и развертывания приложений.

**StatefulSet** - Ресурс Kubernetes, предназначенный для управления приложениями с постоянным состоянием, такими как базы данных.

**Namespace** - Логическое разделение кластера Kubernetes, которое позволяет изолировать ресурсы и поды.

**RBAC** (Role-Based Access Control) - Модель управления доступом, которая позволяет управлять правами доступа пользователей и служб к ресурсам Kubernetes.

# Комманды
## Просмотр тек. конфигурации:
```
kubectl config view
```
## Подключение к кластеру по контексту (~/.kube/config):
```
kubectl config use-context minikube
```
## Подключение к кластеру по конфигурации (~/.kube/cluster.yaml):
```
export KUBECONFIG=~/.kube/cluster.yaml
```
## Просмотр среды кластера:
```
kubectl get namespaces
```
```
kubectl get nodes
```
```
kubectl get pods
```
```
kubectl get services
```
```
kubectl get deployments
```
```
kubectl get secrets
```
```
kubectl get all
```

## Просмотр подробной информации о ресурсе:
```
kubectl describe pod <pod_name>
```
```
kubectl describe service <service_name>
```
```
kubectl describe deployment <deployment_name>
```
## Применение конфигурации из YAML файла:
```
kubectl apply -f <filename.yaml>
```
## Форвард порта пода:
```
kubectl port-forward <pod_name> <port>:<port>
```
## Дефолтный неймспейс:
```
kubectl config set-context --current --namespace=неймспейс
```
## Создание секрета DockerHub:
```
kubectl create secret docker-registry имя-секрета \
--docker-server=https://index.docker.io/v1/ \
--docker-username=имя-пользователя-dockerhub \
--docker-password=пароль-dockerhub \
--docker-email=адрес-электронной-почты
```
## Запуск приложения:
```
kubectl create deployment <deployment_name> --image=<image_name>
```
## Просмотр логов пода:
```
kubectl logs <pod_name>
```
## Исполнение команд внутри пода:
```
kubectl exec -it <pod_name> -- /bin/bash
```
## Удаление ресурсов:
```
kubectl delete pod <pod_name>
```
```
kubectl delete service <service_name>
```
```
kubectl delete deployment <deployment_name>
```
# Helm
## Nginx-controller
### Установка
```
helm install имя-контроллера ingress-nginx --repo https://kubernetes.github.io/ingress-nginx -f путь-к-конфигу-контроллера
```
### Удаление
```
helm uninstall имя-контроллера
```