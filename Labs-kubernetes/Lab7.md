	$ minikube addons enable ingress
	* ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
	You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
	  - Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407
	  - Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407
	  - Using image registry.k8s.io/ingress-nginx/controller:v1.8.1
	* Verifying ingress addon...
	* The 'ingress' addon is enabled


	$ kubectl get pods -n ingress-nginx
	NAME                                        READY   STATUS      RESTARTS   AGE
	ingress-nginx-admission-create-4dk5m        0/1     Completed   0          127m
	ingress-nginx-admission-patch-z74lk         0/1     Completed   0          127m
	ingress-nginx-controller-7799c6795f-77mtn   1/1     Running     0          127m

	$ kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
	$ kubectl expose deployment web --type=NodePort --port=8080
	$ kubectl create deployment web2 --image=gcr.io/google-samples/hello-app:2.0
	$ kubectl expose deployment web2 --type=NodePort --port=8080

	$ k get svc
	NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
	kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          5h42m
	web          NodePort    10.110.175.65    <none>        8080:32112/TCP   77m
	web2         NodePort    10.101.105.184   <none>        8080:32533/TCP   34m
	
	$ k get pod
	NAME                    READY   STATUS    RESTARTS   AGE
	web-548f6458b5-ckdwx    1/1     Running   0          77m
	web2-65959ff6d4-kczj2   1/1     Running   0          33m


Vamos a crear un Ingress basado en path:

	$ vi example-ingress.yaml
	
	apiVersion: networking.k8s.io/v1
	kind: Ingress
	metadata:
	  name: example-ingress
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

	$ k apply -f example-ingress.yaml 

Comprobamos. Puede que tengamos que esperar a ver el ADDRESS

	$ kubectl get ingress
	NAME              CLASS   HOSTS              ADDRESS        PORTS   AGE
	example-ingress   nginx   hello-world.info   192.168.49.2   80      55s

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

Como vemos el Ingress nos permite configurar múltiples apps a las que acceder a través de un mismo dominio, basado en path. También admite instalación de certificados, etc...
