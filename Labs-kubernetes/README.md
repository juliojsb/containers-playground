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

Todos los ficheros de configuración de minikube están en nuestro home, en el directorio oculto **.minikube**
```bash
[jota@jotadev .minikube]$ pwd
/home/jota/.minikube
[jota@jotadev .minikube]$ ls -l
total 28
drwxr-xr-x. 2 jota docker    6 Oct 23  2023 addons
drwxr-xr-x. 4 jota docker   42 Oct 23  2023 cache
-rw-r--r--. 1 jota docker 1111 Oct 23  2023 ca.crt
-rw-------. 1 jota docker 1679 Oct 23  2023 ca.key
-rwxrwxrwx. 1 jota jota   1070 Oct 23  2023 ca.pem
-rwxrwxrwx. 1 jota jota   1115 Oct 23  2023 cert.pem
drwxr-xr-x. 2 jota docker   69 Oct 23  2023 certs
drwxr-xr-x. 2 jota docker    6 Oct 23  2023 config
drwxr-xr-x. 2 jota docker    6 Oct 23  2023 files
-rwxrwxrwx. 1 jota jota   1679 Oct 23  2023 key.pem
drwxr-xr-x. 2 jota docker   45 Oct 23  2023 logs
-rw-------. 1 jota docker    0 Oct 23  2023 machine_client.lock
drwxr-xr-x. 3 jota docker   62 Oct 23  2023 machines
drwxr-xr-x. 3 jota docker   22 Oct 23  2023 profiles
-rw-r--r--. 1 jota docker 1119 Oct 23  2023 proxy-client-ca.crt
-rw-------. 1 jota docker 1675 Oct 23  2023 proxy-client-ca.key
```

### Comandos Minikube

https://minikube.sigs.k8s.io/docs/commands/

### Alternativas a Minikube

* [Kind](https://kind.sigs.k8s.io/)
* [K3S](https://k3s.io/)
* [Kubernets on Docker Desktop](https://docs.docker.com/desktop/kubernetes/)

Tener en cuenta también que existen playgrounds para probar Kubernetes sin tener que instalar nada en nuestro equipo:

* [Killercoda](https://killercoda.com/)
* [Play-with-k8s](https://labs.play-with-k8s.com/)

### Recursos de aprendizaje

* [1000-foot-overview](https://mattias.engineer/k8s/1000-foot-overview/)
