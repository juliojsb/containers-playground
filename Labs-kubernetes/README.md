# Laboratorios Introduccion a Kubernetes

## Requisitos

- Maquina Virtual linux (la misma que usamos para la parte de docker) con al menos 4GB libres en el home del usuario y 2 CPUs
- Conexion a internet

### Instalacion de kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

### Instalación de minikube
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
minikube status
```
Ahora debemos tener acceso a Minikube con kubectl:

```bash
kubectl cluster-info
```
```
Kubernetes master is running at https://192.168.99.100:8443
CoreDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
Probamos a acceder por SSH:
```bash
minikube ssh
```
Accedemos al dashboad:
```bash
minikube dashboard
```
Podemos inicializar Minikube con más de un nodo:
```bash
minikube start --nodes 3 -p multinode-demo
```
O bien ir añadiendo nodos a un clúster existente:
```bash
minikube node add -p multinode-demo
```
NOTA: Si creamos un clúster con la opción `-p` luego hay que incluir el nombre del clúster al interactuar con minikube:
```bash
minikube stop -p multinode-demo
minikube status -p multinode-demo
...
```

### Comandos Minikube

https://minikube.sigs.k8s.io/docs/commands/

### Alternativas a Minikube

* [Kind](https://kind.sigs.k8s.io/)
* [K3S](https://k3s.io/)
* [Kubernets on Docker Desktop](https://docs.docker.com/desktop/kubernetes/)

### Recursos de aprendizaje

* [1000-foot-overview](https://mattias.engineer/k8s/1000-foot-overview/)
