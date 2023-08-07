# Autenticación
Pueden autenticarse con un clúster de Kubernetes usuarios (administradores, desarrolladores, etc..) y aplicaciones. Estas últimas suelen utilizar Service Accounts.

	$ kubectl get sa
	NAME      SECRETS   AGE
	default   0         2d17h

## 1. ServiceAccount de Administración

Vamos a crear una Service Account con permisos de administración en el clúster.

	$ vi sa-admin.yaml
	
	apiVersion: v1
	kind: ServiceAccount
	metadata:
	  name: sa-admin
	  namespace: kube-system
	secrets:
	  - name: secret-sa-admin

Creamos un secret que contendrá el token para esa SA:

	$ vi secret-sa-admin.yaml
	
	apiVersion: v1
	kind: Secret
	metadata:
	  name: secret-sa-admin
	  namespace: kube-system
	  annotations:
	    kubernetes.io/service-account.name: sa-admin
	type: kubernetes.io/service-account-token

Y un ClusterRoleBinding para asignar los permisos del rol cluster-admin a la ServiceAccount sa-admin_

	$ vi crb-sa-admin

	apiVersion: rbac.authorization.k8s.io/v1
	kind: ClusterRoleBinding
	metadata:
	  name: crb-sa-admin
	roleRef:
	  apiGroup: rbac.authorization.k8s.io
	  kind: ClusterRole
	  name: cluster-admin
	subjects:
	  - kind: ServiceAccount
	    name: sa-admin
	    namespace: kube-system
    
Creamos los recursos:

	$ kubectl apply -f sa-admin.yaml 
	$ kubectl apply -f secret-sa-admin.yaml 
	$ kubectl apply -f crb-sa-admin.yaml 

Comprobamos:

	$ kubectl get sa sa-admin -n kube-system
	NAME       SECRETS   AGE
	sa-admin   1         9m17s

	$ kubectl describe sa sa-admin -n kube-system
	Name:                sa-admin
	Namespace:           kube-system
	Labels:              <none>
	Annotations:         <none>
	Image pull secrets:  <none>
	Mountable secrets:   secret-sa-admin
	Tokens:              secret-sa-admin
	Events:              <none>

Intentamos hacer login con el token, debe responder:

	$ TOKEN=$(kubectl get secret secret-sa-admin -n kube-system -o jsonpath='{.data.token}' | base64 --decode)
	$ curl -k -H "Authorization: Bearer $TOKEN" -X GET "https://$(minikube ip):8443/api/v1/nodes"

Configuramos usuario y cambiamos:
 
	$ kubectl config set-credentials sa-admin --token=$TOKEN
	$ kubectl config set-context --current --user=sa-admin

Miramos el kubeconfig, debería haber añadido el usuario:

	$ cat /home/jota/.kube/config
	...
	- name: minikube
	  user:
	    client-certificate: /home/jota/.minikube/profiles/minikube/client.crt
	    client-key: /home/jota/.minikube/profiles/minikube/client.key
	- name: sa-admin
	  user:
	    token: eyJhbGciOi***

Probamos a acceder a recursos, etc...

	$ kubectl get nodes
	NAME       STATUS   ROLES           AGE     VERSION
	minikube   Ready    control-plane   2d17h   v1.27.3


Comprobamos permisos:

	$ kubectl auth can-i '*' '*'
	yes
	$ kubectl auth can-i create pods --all-namespaces
	yes
	$ kubectl auth can-i list deployments
	yes

	$ kubectl auth can-i --list
	Resources                                       Non-Resource URLs                     Resource Names   Verbs
	*.*                                             []                                    []               [*]
	                                                [*]                                   []               [*]
	selfsubjectreviews.authentication.k8s.io        []                                    []               [create]
	selfsubjectaccessreviews.authorization.k8s.io   []                                    []               [create]
	selfsubjectrulesreviews.authorization.k8s.io    []                                    []               [create]
	                                                [/.well-known/openid-configuration]   []               [get]
	                                                [/api/*]                              []               [get]
	                                                [/api]                                []               [get]
	                                                [/apis/*]                             []               [get]
	                                                [/apis]                               []               [get]
	                                                [/healthz]                            []               [get]
	                                                [/healthz]                            []               [get]
	                                                [/livez]                              []               [get]
	                                                [/livez]                              []               [get]
	                                                [/openapi/*]                          []               [get]
	                                                [/openapi]                            []               [get]
	                                                [/openid/v1/jwks]                     []               [get]
	                                                [/readyz]                             []               [get]
	                                                [/readyz]                             []               [get]
	                                                [/version/]                           []               [get]
	                                                [/version/]                           []               [get]
	                                                [/version]                            []               [get]
	                                                [/version]                            []               [get]



Read User  SA

apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa-read
  namespace: kube-system
secrets:
  - name: secret-sa-read
---
apiVersion: v1
kind: Secret
metadata:
  name: secret-sa-read
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: sa-read
type: kubernetes.io/service-account-token
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: sa-read
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: pod-reader
subjects:
  - kind: ServiceAccount
    name: sa-read
    namespace: kube-system
    
Aplicamos:

[jota@srvdev rbac]$ k apply -f sa-read.yaml 
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
serviceaccount/sa-read created
[jota@srvdev rbac]$ k apply -f secret-sa-read.yaml 
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
secret/secret-sa-read created
[jota@srvdev rbac]$ k apply -f cr-pod-reader.yaml 
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
clusterrole.rbac.authorization.k8s.io/pod-reader created
[jota@srvdev rbac]$ k apply -f crb-sa-read.yaml 
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
clusterrolebinding.rbac.authorization.k8s.io/sa-read created

Configuramos el usuario y vemos que sólo podemos listar pods, no nodos, etc...:

[jota@srvdev rbac]$ TOKEN=$(k get secret secret-sa-read -n kube-system -o jsonpath='{.data.token}' | base64 --decode)
[jota@srvdev rbac]$ kubectl config set-credentials sa-read --token=$TOKEN
User "sa-read" set.
[jota@srvdev rbac]$ kubectl config set-context --current --user=sa-read
Context "minikube" modified.
[jota@srvdev rbac]$ k get pods
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
NAME                           READY   STATUS               RESTARTS        AGE
cronjob-hello-28184696-4lvft   0/1     ContainerCannotRun   0               2d14h
cronjob-hello-28184696-4xgtp   0/1     ContainerCannotRun   0               2d14h
cronjob-hello-28184696-67kfv   0/1     ContainerCannotRun   0               2d14h
cronjob-hello-28184696-6f92k   0/1     ContainerCannotRun   0               2d14h
cronjob-hello-28184696-hh9bf   0/1     ContainerCannotRun   0               2d14h
cronjob-hello-28184696-pvct8   0/1     ContainerCannotRun   0               2d14h
cronjob-hello-28184696-pxgdz   0/1     ContainerCannotRun   0               2d14h
cronjob-hello-28188441-t582z   0/1     Completed            0               5m7s
cronjob-hello-28188444-9cxjn   0/1     Completed            0               87s
cronjob-hello-28188445-kqz4f   0/1     Completed            0               67s
cronjob-hello-28188446-gsd9p   0/1     Completed            0               7s
job-hello-1-zgh7x              0/1     Completed            0               2d14h
job-hello-rmjgt                0/1     Completed            0               2d14h
web-548f6458b5-ckdwx           1/1     Running              4 (4m46s ago)   3d14h
web2-65959ff6d4-kczj2          1/1     Running              4 (4m46s ago)   3d13h
[jota@srvdev rbac]$ k get nodes
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
Error from server (Forbidden): nodes is forbidden: User "system:serviceaccount:kube-system:sa-read" cannot list resource "nodes" in API group "" at the cluster scope


Para regenerar el kubeconfig de Minikube, basta con:

1. Parar minikube
minikube stop

2. mover/borrar el fichero actual
mv config config.bkp

3. Rearrancar Minikube
minikube start --cni calico
