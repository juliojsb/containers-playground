# Laboratorios Introduccion a Kubernetes

## Requisitos

- Maquina Virtual linux (la misma que usamos para la parte de docker) con al menos 4GB libres en el home del usuario y 2 CPUs
- Conexion a internet

1. Instalacion de kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

## 3. Instalación de minikube
```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo cp minikube /usr/local/bin/ && rm minikube
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
ssh-keygen -t rsa -b 4096
minikube start --driver=docker
```
Una vez arrancado comprobamos el estado:
```bash
$ minikube status
```
## 4. En este punto ya se puede tener acceso a minikube con kubectl
```bash
kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443
CoreDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

## 5. Para acceder a la VM de minikube
```bash
minikube ssh
```

## 6. Acceso al dashboard de minikube
```bash
minikube dashboard
```
## 7. Cluster multinodo

Podemos inicializar minikube con más de un nodo:
```bash
minikube start --nodes 3 -p multinode-demo
bash
O bien ir añadiendo nodos a un clúster existente:
```bash
$ minikube node add -p multinode-demo
```
NOTA: Si creamos un clúster con la opción `-p` luego hay que incluir el nombre del clúster al interactuar con minikube:
```bash
$ minikube stop -p multinode-demo
$ minikube status -p multinode-demo
...
```

## 8. Comandos Minikube

https://minikube.sigs.k8s.io/docs/commands/
