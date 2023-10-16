# Troubleshooting

Kubernetes es una plataforma compleja y a la hora de realizar troubleshooting es recomendable segmentar el análisis a distintos niveles.

* ¿Se trata de problema de aplicación?
* ¿Estado general de la infraestructura?
* ¿Los nodos están OnPrem? Si es así, ¿qué virtualizador se está utilizando? (RHV, Vmware, HyperV...)
* ¿Los nodos están en un proveedor Cloud? ¿Azure, GCP, AWS...? Mirar matriz de responsabilidades

## Clúster

Información del clúster:

	$ kubectl get nodes
	$ kubectl cluster-info
	$ kubectl cluster-info dump

Dentro del nodo, podemos revisar los logs de los distintos componentes del control plane:

En Minikube encontraremos los logs en `/var/log/pods`

	$ /var/log/pods/kube-system_etcd-minikube_8af0e85a28544808d52bb7c47ad824ed/17.log
 	$ /var/log/pods/kube-system_kube-apiserver-minikube_4e275e35949ad3fdfeb753c1099308e7/20.log
  	...

En una instalación de Kubernetes encontraremos esos logs en `/var/log`

	$ /var/log/kube-apiserver.log
	$ /var/log/kubelet.log
	...

Estado de servicios, recursos, etc...

	$ top
	$ df -h
	$ free -h
	$ systemctl status kubelet
	$ journalctl -u kubelet
	$ Certificados en 

¿Certificados?
* Kubelet
```
/var/lib/kubelet/pki
```
* ETCD, API Server...
```
/etc/kubernetes/pki
```
TIP: revisar certificados referenciados desde los servicios en `/etc/systemd/system/`

Logs e info de aplicación, servicios...
	
	$ kubectl logs [pod-name]
	$ kubectl describe pod [pod-name]
	$ kubectl get endpoints [service-name]

## Aplicaciones

Acceder a apps para comprobar funcionalidad:

Mantenimiento:

	$ kubectl drain minikube --ignore-daemonsets --delete-emptydir-data --force
	$ kubectl cordon minikube
	$ kubectl uncordon minikube

Recursos prácticos de troubleshooting en K8S:

* [Troubleshooting flow](https://learnk8s.io/troubleshooting-deployments)
