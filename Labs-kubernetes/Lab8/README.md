# Instrucciones Laboratorio 9 - Kubernetes - NetworkPolicies

La instalación por defecto de minikube utiliza el CNI Kindnet y no soporta NetworkPolicies. Tenemos que arrancar con un plugin que soporte esta funcionalidad.

Plugins que soportan NetworkPolicies:
- Kube-router
- Calico
- Romana
- Weavenet
	
Plugins que NO soportan NetworkPolicies:
- Flannel
- Kindnet

Paramos y borramos la instancia de minikube para arrancar en limpio con el plugin Calico:
```bash
$ minikube stop
$ minikube delete
```
Arrancamos con la opción `--cni calico`
```bash
$ minikube start --cni calico
```
Debemos ver que carga el plugin:
```bash
* minikube v1.31.1 on Centos 7.9.2009 (amd64)
* Automatically selected the docker driver. Other choices: none, ssh
* Using Docker driver with root privileges
* Starting control plane node minikube in cluster minikube
* Pulling base image ...
* Creating docker container (CPUs=2, Memory=2200MB) ...
* Preparing Kubernetes v1.27.3 on Docker 24.0.4 ...
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Configuring Calico (Container Networking Interface) ...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Verifying Kubernetes components...
* Enabled addons: storage-provisioner, default-storageclass
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
Creamos dos namespaces y desplegamos un Nginx en cada uno de ellos:
```bash
$ kubectl create ns ns1
$ kubectl create ns ns2
$ kubectl run nginx1 --image=nginx --restart=Never --labels=app=nginx -n ns1
$ kubectl run nginx2 --image=nginx --restart=Never --labels=app=nginx -n ns2
```
Revisamos los pods. Vamos a necesitar la IP interna con la que levantan:
```bash
$ kubectl get pod -n ns1 -o wide
NAME     READY   STATUS    RESTARTS   AGE   IP              NODE       NOMINATED NODE   READINESS GATES
nginx1   1/1     Running   0          29s   10.244.120.67   minikube   <none>           <none>

$ kubectl get pod -n ns2 -o wide
NAME     READY   STATUS    RESTARTS   AGE   IP              NODE       NOMINATED NODE   READINESS GATES
nginx2   1/1     Running   0          30s   10.244.120.68   minikube   <none>           <none>
```
Creamos una **NetworkPolicy** que bloquea por defecto todo el tráfico entrante y saliente del namespace ns-1:
```yaml
$ vi netpol-ns1.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: ns1
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  - Egress

$ kubectl apply -f netpol-ns1.yaml
$ k get netpol -n ns1
NAME                   POD-SELECTOR   AGE
default-deny	          app=nginx      99m
```
Entramos en el pod de Nginx del namespace ns2:
```bash
$ kubectl exec -it nginx2 -n ns2 sh
# curl http://10.244.120.67
```
Vemos que no podemos acceder al pod de Nginx del namespace ns1

Ahora vamos a modificar la NetworkPolicy anterior y permitir acceso tráfico en el puerto 80. En el spec añadimos la configuración para el tráfico entrante:
```yaml
$ kubectl edit netpol default-deny -n ns1

...
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - ports:
    - port: 80
```
Guardamos y salimos.

Volvemos a entrar al pod de Nginx en el namespace ns2 y hacemos curl. Vemos que ahora podemos acceder al pod de Nginx en ns1:
```bash
$ kubectl exec -it nginx2 -n ns2 sh
# curl http://10.244.120.67
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>
```
	<p><em>Thank you for using nginx.</em></p>
	</body>
	</html>

