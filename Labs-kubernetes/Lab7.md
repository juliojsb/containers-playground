 # Elementos del control plane

	$ kubectl get pods -n kube-system
 	NAME                               READY   STATUS    RESTARTS   AGE
	coredns-558bd4d5db-97fvq           1/1     Running   0          119d
	etcd-minikube                      1/1     Running   0          119d
	kindnet-96tsb                      1/1     Running   0          119d
	kube-apiserver-minikube            1/1     Running   0          119d
	kube-controller-manager-minikube   1/1     Running   0          119d
	kube-proxy-npdhr                   1/1     Running   0          119d
	kube-scheduler-minikube            1/1     Running   0          119d
	metrics-server-77c99ccb96-cvdjb    1/1     Running   0          12d
	storage-provisioner                1/1     Running   1          119d

	$ kubectl get nodes
	NAME       STATUS   ROLES                  AGE    VERSION
	minikube   Ready    control-plane,master   119d   v1.21.2

Los roles no son más que labels. Podemos verlos, junto con más detalles del nodo, con `describe`:

	$ kubectl describe node minikube
