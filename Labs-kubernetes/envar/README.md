apiVersion: v1
kind: Pod
metadata:
 name: pod-envar
  labels:
    purpose: ejemplo-envar
spec:
  containers:
  - name: pod-envar-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"

 

$ kubectl apply -f podvars.yaml

$ kubectl get po
NAME                    READY   STATUS    RESTARTS      AGE
pod-envar               1/1     Running   0             33s

Comprobamos las variables de entorno entrando al pod:

$ kubectl exec -it pod-envar -- bash
root@pod-envar:/# env | grep DEMO
DEMO_FAREWELL=Such a sweet sorrow
DEMO_GREETING=Hello from the environment

envFrom se utiliza para definir variables de entorno desde un ConfigMap o Secret, lo que veremos en Labs posteriores
