# Ingress
En este laboratorio vamos a crear un recurso de tipo Ingress y a dirigir tráfico a los pods a través de él de dos maneras:

- Routing basado en path: desde un mismo dominio, accederemos a una u otra aplicación en función del contexto (path)
- Routing basado en hostname: en función del dominio, accederemos a una u otra aplicación.

## Habilitar Ingress en Minikube

En primer lugar, habilitamos el addon en Minikube:
    
 	$ minikube addons enable ingress
	* ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
	You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
	  - Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407
	  - Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407
	  - Using image registry.k8s.io/ingress-nginx/controller:v1.8.1
	* Verifying ingress addon...
	* The 'ingress' addon is enabled

Comprobamos que se los recursos se han desplegado correctamente:

	$ kubectl get pods -n ingress-nginx
	NAME                                        READY   STATUS      RESTARTS   AGE
	ingress-nginx-admission-create-4dk5m        0/1     Completed   0          127m
	ingress-nginx-admission-patch-z74lk         0/1     Completed   0          127m
	ingress-nginx-controller-7799c6795f-77mtn   1/1     Running     0          127m

## Routing basado en path

Creamos dos aplicaciones de prueba, y exponemos los pods mediante un servicio de tipo NodePort:

	$ kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
	$ kubectl expose deployment web --type=NodePort --port=8080
	$ kubectl create deployment web2 --image=gcr.io/google-samples/hello-app:2.0
	$ kubectl expose deployment web2 --type=NodePort --port=8080

Comprobamos que se han creado los servicios:

	$ kubectl get svc
	NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
	kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          5h42m
	web          NodePort    10.110.175.65    <none>        8080:32112/TCP   77m
	web2         NodePort    10.101.105.184   <none>        8080:32533/TCP   34m

Comprobamos que se han desplegado los pods:
 
	$ kubectl get pod
	NAME                    READY   STATUS    RESTARTS   AGE
	web-548f6458b5-ckdwx    1/1     Running   0          77m
	web2-65959ff6d4-kczj2   1/1     Running   0          33m


Vamos a crear un Ingress basado en path. Este recurso Ingress, contiene la configuración para acceder a ambas aplicaciones. Tendrán un hostname común `hello-world.info` y se diferencian por el contexto de app:

	$ vi ingress-path.yaml
	
	apiVersion: networking.k8s.io/v1
	kind: Ingress
	metadata:
	  name: ingress-path
	  annotations:
	    nginx.ingress.kubernetes.io/rewrite-target: /$1
	spec:
	  rules:
	    - host: hello-world.info
	      http:
	        paths:
	          - path: /v1
	            pathType: Prefix
	            backend:
	              service:
	                name: web
	                port:
	                  number: 8080
	          - path: /v2
	            pathType: Prefix
	            backend:
	              service:
	                name: web2
	                port:
	                  number: 8080

	$ kubectl apply -f ingress-path.yaml 

Comprobamos. Puede que tengamos que esperar a ver el ADDRESS en el Ingress creado:

	$ kubectl get ingress
	NAME              CLASS   HOSTS              ADDRESS        PORTS   AGE
	ingress-path      nginx   hello-world.info   192.168.49.2   80      55s

Accedemos a las aplicaciones mediante curl:

	$ curl --resolve "hello-world.info:80:$( minikube ip )" -i http://hello-world.info/v1
	HTTP/1.1 200 OK
	Date: Wed, 02 Aug 2023 18:18:51 GMT
	Content-Type: text/plain; charset=utf-8
	Content-Length: 60
	Connection: keep-alive
	
	Hello, world!
	Version: 1.0.0
	Hostname: web-548f6458b5-ckdwx
	
	$ curl --resolve "hello-world.info:80:$( minikube ip )" -i http://hello-world.info/v2
	HTTP/1.1 200 OK
	Date: Wed, 02 Aug 2023 18:16:57 GMT
	Content-Type: text/plain; charset=utf-8
	Content-Length: 61
	Connection: keep-alive
	
	Hello, world!
	Version: 2.0.0
	Hostname: web2-65959ff6d4-kczj2

Vemos que el hostname es el nombre del pod

Como vemos el Ingress nos permite configurar múltiples apps a las que acceder a través de un mismo dominio, basado en path.

## Routing basado en hostname

En este ejemplo vamos a crear un Ingress basado en hostname. Según accedamos a un hostname u otro, iremos a una aplicación. No hace falta volver a desplegar las aplicaciones, vamos a reutilizar las que desplegamos en el ejemplo anterior:

	$ vi ingress-hostname.yaml
	
	apiVersion: networking.k8s.io/v1
	kind: Ingress
	metadata:
	  name: ingress-hostname
	  annotations:
	    nginx.ingress.kubernetes.io/rewrite-target: /$1
	spec:
	  rules:
	    - host: hello-world-v1.info
	      http:
	        paths:
	          - path: /
	            pathType: Prefix
	            backend:
	              service:
	                name: web
	                port:
	                  number: 8080
	    - host: hello-world-v2.info
	      http:
	        paths:
	          - path: /
	            pathType: Prefix
	            backend:
	              service:
	                name: web2
	                port:
	                  number: 8080
	
	$ kubectl apply -f ingress-hostname.yaml

Comprobamos que se ha creado el recurso:

	$ kubectl get ingress
	NAME               CLASS   HOSTS                                     ADDRESS        PORTS   AGE
	ingress-path       nginx   hello-world.info                          192.168.49.2   80      8d
	ingress-hostname   nginx   hello-world-v1.info,hello-world-v2.info   192.168.49.2   80      24s

Accedemos con curl a cada aplicación:

	$ curl --resolve "hello-world-v1.info:80:$( minikube ip )" -i http://hello-world-v1.info
	HTTP/1.1 200 OK
	Date: Thu, 10 Aug 2023 11:10:40 GMT
	Content-Type: text/plain; charset=utf-8
	Content-Length: 60
	Connection: keep-alive
	
	Hello, world!
	Version: 1.0.0
	Hostname: web-548f6458b5-x6wvv
	
	$ curl --resolve "hello-world-v2.info:80:$( minikube ip )" -i http://hello-world-v2.info
	HTTP/1.1 200 OK
	Date: Thu, 10 Aug 2023 11:10:50 GMT
	Content-Type: text/plain; charset=utf-8
	Content-Length: 61
	Connection: keep-alive
	
	Hello, world!
	Version: 2.0.0
	Hostname: web2-65959ff6d4-mkp7b
