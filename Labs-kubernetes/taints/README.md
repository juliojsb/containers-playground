## Taints & Tolerations

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
5. Revisamos que el pod ahora se ejecuta:
```bash
$ kubectl get pod -n my-nginx
NAME     READY   STATUS    RESTARTS   AGE
nginx1   1/1     Running   0          5m49s
```
Borramos el namespace:

	$ kubectl delete ns my-nginx
