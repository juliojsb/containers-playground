# Laboratorio 6 - Services

Los servicios permiten acceder a los distintos pods que tenemos desplegados a nuestro cluster. Tenemos:

* **ClusterIP:** sólo accesible internamente en el clúster.
* **NodePort:** podemos acceder externamente a las aplicaciones con la IP del nodo y el nodePort especificado.
* **LoadBalancer:** permite acceder a aplicaciones de forma externa si está integrado con un proveedor que asigne IPs externas al recurso LoadBalancer de Kubernetes. Existen alternativas para asignar IP's externas en entornos OnPrem como MetalLB.

Adicionalmente, podremos utilizar [port-forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) para acceder a nuestras aplicaciones de forma puntual (debugging, test...)

Ejemplo:
```bash
kubectl port-forward redis-master-461e559876-218fy 7000:6379
```
Accedemos desde localhost:7000 (por ejemplo, desde nuestro navegador) al pod de Redis Master que utiliza el puerto (interno del pod) 6379

Para este Lab, vamos a desplegar una aplicación de prueba:
```yaml
$ vi deploy-nginx.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
  labels:
    app: my-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-nginx
  template:
    metadata:
      labels:
  app: my-nginx
    spec:
      containers:
      - name: my-nginx
  image: nginx:alpine
  ports:
  - containerPort: 80
  resources:
    limits:
      memory: "128Mi" 
      cpu: "200m"

$ kubectl apply -f deploy-nginx.yaml
```
## ClusterIP

En servicio tipo ClusterIP sólo es accesible internamente en el clúster:
```yaml
$ vi svc-nginx-clusterip.yaml

apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  type: ClusterIP
  ports:
    - targetport: 80
      port: 80
  selector:
     app: my-nginx

$ kubectl apply -f svc-nginx-clusterip.yaml
```
Comprobamos la IP interna que le ha asignado al servicio:
```
$ kubectl get svc
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-clusterip   ClusterIP   10.108.36.45    <none>        8080/TCP       22s
```
Entramos a un pod de Nginx y hacemos una request a la IP de ClusterIP:
```
$ kubectl get pod
NAME                        READY   STATUS    RESTARTS   AGE
my-nginx-5bb9b897c8-6gnt2   1/1     Running   0          13m
my-nginx-5bb9b897c8-tk4vv   1/1     Running   0          13m

$ k exec -it my-nginx-5bb9b897c8-6gnt2 -- sh
/ # curl http://10.108.36.45:8080
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

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
## NodePort

Con un servicio NodePort podemos acceder a las aplicaciones con la IP del nodo y el nodePort especificado:
```yaml
$ vi svc-nginx-nodeport.yaml

apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: Nodeport
  ports:
    - targetport: 80
      port: 80
      nodePort: 30008
  selector:
    app: my-nginx


$ kubectl apply -f svc-nginx-nodeport.yaml
```
Comprobamos la IP interna que le ha asignado al servicio:
```
$ kubectl get svc
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-nodeport    NodePort    10.96.167.146   <none>        80:31000/TCP   15s
```
Sin embargo, al ser del tipo NodePort podemos acceder a través del puerto 31000 y la IP del nodo:
```
$ kubectl get node -o wide
NAME       STATUS   ROLES                  AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION                CONTAINER-RUNTIME
minikube   Ready    control-plane,master   6d    v1.21.2   192.168.49.2   <none>        Ubuntu 20.04.2 LTS   3.10.0-1160.81.1.el7.x86_64   docker://20.10.7
```
Comprobamos con curl:
```bash
$ curl http://192.168.49.2:31000
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

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
## LoadBalancer

Un servicio LoadBalancer permite acceder a aplicaciones de forma externa si está integrado con un proveedor que asigne IPs externas al recurso LoadBalancer de Kubernetes:
```yaml
$ vi svc-nginx-loadbalancer.yaml

apiVersion: v1
kind: Service
metadata:
 name: nginx-loadbalancer
spec:
 type: LoadBalancer
 selector:
    app: my-nginx
 ports:
  - name: "80"
    port: 80
    targetPort: 80
  - name: "443"
    port: 443
    targetPort: 443

$ kubectl apply -f svc-nginx-loadbalancer.yaml
```
Comprobamos la IP asignada. En un proveedor cloud, se asignaría una IP externa para poder acceder al servicio. En este caso, podemos comprobar tanto la IP interna como a través del nodo:
```
$ kubectl get svc
NAME                TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)                      AGE
nginx-loadbalancer 	LoadBalancer   10.109.148.110   <none>			   80:31333/TCP,443:31044/TCP   7d22h
```
Comprobamos acceso a través de la ClusterIP:
```bash
$ kubectl exec -it my-nginx-55c8bc54f9-l42h7 sh
/ # curl http://10.109.148.110
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

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
/ # exit
```
Comprobamos acceso a través del nodo:
```bash
$ curl http://192.168.49.2:31333
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

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
## Acceso externo entornos OnPrem
Para entornos cloud, los principales proveedores como AWS, Azure o GCP ofrecen integración de Kubernetes con su API para poder proveer de IPs externas a los servicios y poder acceder externamente a las aplicaciones.

A diferencia de estos proveedores, en entornos OnPrem podemos valorar el uso de MetalLB que actúa como un servidor DHCP dentro del clúster de Kubernetes, asignando IPs externas a los servicios que lo requieran:

https://metallb.universe.tf/

MetalLB además ofrece integración con protocolo BGP, por lo que puede integrarse más fácilmente en nuestra red.

Otra solución consiste en apuntar desde un F5, HAProxy o httpd/nginx intermediarios a servicios de tipo NodePort para poder acceder externamente a las aplicaciones de Kubernetes.
