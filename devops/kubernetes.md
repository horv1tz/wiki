---
description: Шпаргалка по Kubernetes — kubectl, манифесты, Helm
---

# Kubernetes

**Kubernetes (k8s)** — платформа оркестрации контейнеров: запускает, масштабирует и перезапускает контейнеризированные приложения на кластере серверов. **kubectl** — основной CLI для управления кластером.

## 1. Установка kubectl

```bash
# Debian/Ubuntu
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update && sudo apt install kubectl
```

Для локальных экспериментов поднимите кластер: [minikube](https://minikube.sigs.k8s.io/), [kind](https://kind.sigs.k8s.io/) или легковесный [k3s](https://k3s.io/):

```bash
# k3s на сервере — полноценный k8s в один бинарник
curl -sfL https://get.k3s.io | sh -
sudo k3s kubectl get nodes
```

## 2. Контексты и подключение

```bash
kubectl config get-contexts           # список контекстов (кластер + пользователь)
kubectl config use-context <имя>      # переключение контекста
kubectl cluster-info                  # информация о кластере
kubectl get nodes                     # список узлов
```

## 3. Основные команды kubectl

```bash
# Объекты
kubectl get pods                      # поды в текущем namespace
kubectl get pods -A                   # поды во всех namespace
kubectl get deployments,services      # несколько типов сразу
kubectl describe pod <имя>            # подробности и события объекта
kubectl delete pod <имя>

# Создание и применение манифестов
kubectl apply -f manifest.yaml        # создать/обновить объекты из файла
kubectl apply -f ./manifests/         # применить всю директорию
kubectl diff -f manifest.yaml         # что изменится после apply
kubectl delete -f manifest.yaml

# Логи и отладка
kubectl logs <под>                    # логи
kubectl logs -f <под> -c <контейнер>  # follow, конкретный контейнер
kubectl exec -it <под> -- /bin/sh     # shell внутри контейнера
kubectl port-forward svc/<сервис> 8080:80   # проброс порта на локальную машину
kubectl run debug --rm -it --image=alpine -- sh   # временный отладочный под

# Deployments
kubectl rollout status deployment/<имя>
kubectl rollout history deployment/<имя>
kubectl rollout undo deployment/<имя> # откат на предыдущую ревизию
kubectl scale deployment/<имя> --replicas=3
```

## 4. Базовый манифест

`deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: nginx:1.27
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 256Mi
---
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

```bash
kubectl apply -f deployment.yaml
kubectl get pods -l app=web
```

## 5. Концепции в двух словах

* **Pod** — минимальная единица: один или несколько контейнеров с общей сетью
* **Deployment** — управляет репликами подов и rolling-обновлениями
* **Service** — стабильный сетевой адрес для группы подов (ClusterIP / NodePort / LoadBalancer)
* **Ingress** — HTTP-маршрутизация снаружи внутрь кластера (нужен ingress-контроллер, например ingress-nginx)
* **ConfigMap / Secret** — конфигурация и секреты, отделённые от образа
* **Namespace** — логическое разделение ресурсов кластера
* **PersistentVolumeClaim** — запрос постоянного хранилища для пода

## 6. Helm — пакетный менеджер Kubernetes

```bash
# Установка Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Основные команды
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx                  # поиск чарта
helm install my-nginx bitnami/nginx     # установка
helm list                               # установленные релизы
helm upgrade my-nginx bitnami/nginx     # обновление
helm rollback my-nginx                  # откат
helm uninstall my-nginx                 # удаление

# Переопределение значений чарта
helm install my-nginx bitnami/nginx -f values.yaml
helm show values bitnami/nginx          # посмотреть доступные параметры
```

## 7. Полезные ссылки

* [Документация Kubernetes](https://kubernetes.io/ru/docs/)
* [Шпаргалка kubectl](https://kubernetes.io/docs/reference/kubectl/quick-reference/)
* [Документация Helm](https://helm.sh/docs/)
