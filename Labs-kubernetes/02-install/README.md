# Architecture

## ETCD

|Cluster Size|Majority|Failure Tolerance|
|---|---|---|
|1|1|0|
|2|2|0|
|3|2|1|
|4|3|1|
|5|3|2|
|6|4|2|
|7|4|3|
|8|5|3|
|9|5|4|

Opción más común y mínima: 3 control plane

Los control plane NO deben ser workers, las cargas de trabajo deben ir en nodos dedicados especialmente en entornos productivos.

Alta disponibilidad:

* [Azure](https://learn.microsoft.com/en-us/azure/aks/availability-zones): distribución de nodos en distintas Availability Zones


* [GCP](https://cloud.google.com/kubernetes-engine/docs/concepts/regional-clusters): distintas zonas en regiones


* [AWS](https://docs.aws.amazon.com/eks/latest/userguide/disaster-recovery-resiliency.html): distintas availability zones en regiones


En OnPrem o si se dispone únicamente de 2 sites (CPDs):

* Para evitar pérdida de quorum eventual y problemas con los control plane, es preferible diseñar estrategias de DR teniendo todo el clúster en un CPD y en caso de desastre, levantarlo en respaldo. 
* Otra opción es tener dos clústers en HA con uno activo y otro standby, y sólo dirigir tráfico al standby en caso necesario.
* Las soluciones activo-activo dependerán en gran parte de aplicaciones y almacenamiento utilizado por lo que son aún más complejas de implementar.

## Formas de desplegar Kubernetes

* Kubernetes vanilla
  * [Minikube](https://minikube.sigs.k8s.io/docs/) (Testing, local)
  * [Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) (OnPrem & Cloud)
  * [Kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/) (OnPrem & Cloud)
  * [Kubespray](https://github.com/kubernetes-sigs/kubespray) (Declarativo con YAML; Ansible, OnPrem & Cloud)
  * [Kops](https://kops.sigs.k8s.io/) (Declarativo con YAML; Cloud)

* Kubernetes gestionados en Cloud:
  * [EKS](https://aws.amazon.com/eks/)
  * [AKS](https://azure.microsoft.com/en-us/products/kubernetes-service)
  * [GKE](https://cloud.google.com/kubernetes-engine)
  * [LKE](https://www.linode.com/products/kubernetes/)
  * [DOKS](https://www.digitalocean.com/products/kubernetes)
