# Instrucciones Laboratorio 10 - Kubernetes - Troubleshooting

Kubernetes es una plataforma compleja y a la hora de realizar troubleshooting es recomendable segmentar el análisis a distintos niveles.

* ¿Se trata de problema de aplicación?
* ¿Estado general de la infraestructura?
* ....

Información del clúster:

	$ kubectl get nodes
	$ kubectl cluster-info
	$ kubectl cluster-info dump

Dentro del nodo, logs de componentes:

	$ /var/log/kube-apiserver.log
	$ /var/log/kubelet.log

Estado de servicios, recursos, etc...

	$ top
	$ df -h
	$ free -h
	$ Check status of Kubelet service with systemctl
	$ Check logs with journalctl -u kubelet
	$ Check kubelet certificates in /var/lib/kubelet

Logs e info de aplicación, servicios...
	
	$ kubectl logs [pod-name]
	$ kubectl describe pod [pod-name]
	$ kubectl get endpoints [service-name]

Acceder a apps para comprobar funcionalidad:

Mantenimiento:

	$ kubectl drain minikube --ignore-daemonsets --delete-emptydir-data --force
	$ kubectl cordon minikube
	$ kubectl uncordon minikube
