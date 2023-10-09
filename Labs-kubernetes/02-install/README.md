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

Formas de desplegar Kubernetes

* Kubernetes vanilla
  * [Minikube](https://minikube.sigs.k8s.io/docs/) (Testing, local)
  * [Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
  * [Kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/) (OnPrem & Cloud)
  * [Kubespray](https://github.com/kubernetes-sigs/kubespray) (Declarativo con YAML; Ansible, OnPrem & Cloud)
  * [Kops](https://kops.sigs.k8s.io/) (Declarativo con YAML; Cloud)

* Kubernetes gestionados en Cloud:
  * [EKS](https://aws.amazon.com/eks/)
  * [AKS](https://azure.microsoft.com/en-us/products/kubernetes-service)
  * [GKE](https://cloud.google.com/kubernetes-engine)
  * [LKE](https://www.linode.com/products/kubernetes/)
  * [DOKS](https://www.digitalocean.com/products/kubernetes)
