# greenlama_microservices
greenlama microservices repository

## ДЗ №15

- Подготовил инсталляцию Gitlab CI
- Подготовил репозиторий с кодом приложения
- Описал для приложения этапы пайплайна
- Определил окружения

## ДЗ №16
- Prometheus: запуск, конфигурация, знакомство с Web UI
- Мониторинг состояния микросервисов
- Сбор метрик хоста с использованием экспортера

Ссылки на docker hub:
https://hub.docker.com/repository/docker/greenlama/prometheus
https://hub.docker.com/repository/docker/greenlama/post
https://hub.docker.com/repository/docker/greenlama/comment
https://hub.docker.com/repository/docker/greenlama/ui

## ДЗ №17
- Подготовка окружения
- Логирование Docker-контейнеров
- Сбор неструктурированных логов
- Визуализация логов
- Сбор структурированных логов
- Распределенный трейсинг

## ДЗ №18
- Разобрал на практике все компоненты Kubernetes, развернув их вручную, используя kubeadm
- Добавил манифесты приложения

На мастере выполняем:
```
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install docker-ce=5:19.03.15~3-0~ubuntu-focal docker-ce-cli=5:19.03.15~3-0~ubuntu-focal containerd.io docker-compose-plugin
sudo apt-get install -y kubelet=1.19.16-00 kubeadm=1.19.16-00 kubectl=1.19.16-00

# Указываем свой внешний IP адрес мастера
sudo kubeadm init --apiserver-cert-extra-sans=51.250.8.74 --apiserver-advertise-address=0.0.0.0 --control-plane-endpoint=51.250.8.74 --pod-network-cidr=10.244.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

На воркере выполняем:
```
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release -y
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install docker-ce=5:19.03.15~3-0~ubuntu-focal docker-ce-cli=5:19.03.15~3-0~ubuntu-focal containerd.io docker-compose-plugin -y
sudo apt-get install -y kubelet=1.19.16-00 kubeadm=1.19.16-00 kubectl=1.19.16-00 -y

# Строка для подключения воркера, полученная после инициализации kubeadm на мастере
sudo kubeadm join 51.250.8.74:6443 --token pvhwc6.47dw060e28y5qawn \
    --discovery-token-ca-cert-hash sha256:71c63a3e373751df46de971d4b6616ae163496d98a2001113050b42937e6cbda
```
После добавления воркера на мастере выполняем:
```
curl https://docs.projectcalico.org/v3.20/manifests/calico.yaml -o calico.yaml
nano  calico.yaml
- name: CALICO_IPV4POOL_CIDR
  value: "10.244.0.0/16"
kubectl apply -f calico.yaml
```

Проверяем ноды на мастере:
```
kubectl get nodes
```

Для управления кластером со своего компа копируем с мастера файл ```/etc/kubernetes/admin.conf```. Запуск на своем компе выполняется так:
```
kubectl --kubeconfig ./admin.conf get nodes
```
