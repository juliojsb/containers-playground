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

Formas de desplegar Kubernetes "vanilla"

* Minikube (testing)
* Hard Way
* Kubeadm
* Kubespray
* Kops

# Minikube

https://minikube.sigs.k8s.io/docs/

# Kubeadm

...
