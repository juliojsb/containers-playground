# Taints & Tolerations

Existen taints que podemos añadir de forma manual y otros que se añaden de forma dinámica. Por ejemplo, cuando un nodo se está quedando sin memoria y el kubelet reporta la condición `memory-pressure` ¿qué hace Kubernetes para impedir que se ejecuten cargas de trabajo en ese nodo? Añade un taint de forma temporal mientras persista esa condición:
```
node.kubernetes.io/memory-pressure
```
Existen los siguientes taints soportados "de serie" que Kubernetes puede añadir a los nodos:
```
node.kubernetes.io/not-ready
node.kubernetes.io/unreachable
node.kubernetes.io/memory-pressure
node.kubernetes.io/disk-pressure
node.kubernetes.io/pid-pressure
node.kubernetes.io/network-unavailable
node.kubernetes.io/unschedulable
node.cloudprovider.kubernetes.io/uninitialized
```
A continuación, vamos a evitar que un pod se ejecute en un nodo al no contar con el toleration necesario.

1. Ejecutamos de nuevo el pod nginx1:
```bash
$ kubectl create namespace my-nginx
$ kubectl run nginx1 --image=nginx --restart=Never --labels=app=v1 -n my-nginx
```

2. Creamos taints para el nodo minikube:
```bash
$ kubectl taint nodes minikube type=specialnode:NoSchedule
$ kubectl taint nodes minikube type=specialnode:NoExecute
```
En este caso `type` es la clave y `specialnode` el valor.

3. Comprobamos que el pod no se puede ejecutar:
```bash
$ kubectl get pod -n my-nginx
NAME     READY   STATUS    RESTARTS   AGE
nginx1   0/1     Pending   0          10s

$ kubectl describe pod nginx1 -n my-nginx
...
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  24s   default-scheduler  0/1 nodes are available: 1 node(s) had untolerated taint {type: specialnode}. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling..
```
4. Añadimos toleration al pod:
```bash
$ kubectl edit pod nginx1 -n my-nginx
  tolerations:
...
  - effect: NoSchedule
    key: type
    operator: Equal
    value: "specialnode"
  - effect: NoExecute
    key: type
    operator: Equal
    value: "specialnode"
    tolerationSeconds: 300
```
5. Revisamos que el pod ahora se puede ejecutar en el nodo:
```bash
$ kubectl get pod -n my-nginx
NAME     READY   STATUS    RESTARTS   AGE
nginx1   1/1     Running   0          5m49s
```
Borramos el namespace:
```bash
$ kubectl delete ns my-nginx
```
