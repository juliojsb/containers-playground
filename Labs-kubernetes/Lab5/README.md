# Instrucciones Laboratorio 5 - Kubernetes - HPA
En este laboratorio vamos a crear un HPA y ver cómo aumenta y disminuye el nº de réplicas en función de la carga de la aplicación

Creamos el deployment:
```bash
$ vi deployment-apache.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apache
spec:
  selector:
    matchLabels:
      run: apache
  template:
    metadata:
      labels:
        run: apache
    spec:
      containers:
      - name: apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
```
Creamos un servicio para Apache:
```bash
$ vi svc-apache.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: apache
  labels:
    run: apache
spec:
  ports:
  - port: 80
  selector:
    run: apache
```
```bash
$ kubectl apply -f deployment-apache.yaml
$ kubectl apply -f svc-apache.yaml
```
Comprobamos que se han creado los recursos:
```bash
$ kubectl get pod
$ kubectl get deploy
$ kubectl get svc
```
Creamos el HPA:
```bash
$ vi hpa-apache.yaml
```
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  creationTimestamp: null
  name: hpa-apache
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: apache
  targetCPUUtilizationPercentage: 50
```
```bash
$ kubectl apply -f hpa-apache.yaml
```

Vamos a generar carga. Tendremos que crear tráfico artificial en el pod de Apache (Tip: aplicación útil para generar tráfico https://github.com/rakyll/hey). ¿Cómo lo haremos? A través del servicio creado anteriormente. Al ser ClusterIP, es accesible internamente desde el clúster. Por tanto, desde otro pod:
```bash
$ kubectl get svc
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
apache        ClusterIP   10.107.43.68    <none>        80/TCP    82s
```
Creamos un pod provisional de generación de carga apuntando a la ClusterIP del servicio de Apache:
```bash
$ kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://10.107.43.68; done"
```
Observamos el HPA:
```bash
$ watch -n1 "kubectl get hpa"
```
O bien:
```bash
$ kubectl get hpa hpa-apache --watch
```

Vemos como se crean réplicas a medida que aumenta la carga:
```bash
$ kubectl get hpa
NAME          REFERENCE                TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
hpa-apache    Deployment/apache        250%/50%   1         10        4          2m24s
nginx-curso   Deployment/nginx-curso   4%/50%     1         10        1          33m

$ kubectl get pod
NAME                           READY   STATUS    RESTARTS   AGE
apache-5c9cdffd47-8hz2c        0/1     Pending   0          1s
apache-5c9cdffd47-dg8jn        0/1     Pending   0          31s
apache-5c9cdffd47-hmkx6        1/1     Running   0          46s
apache-5c9cdffd47-jshv7        0/1     Pending   0          1s
apache-5c9cdffd47-lxq5b        1/1     Running   0          46s
apache-5c9cdffd47-mb8m9        1/1     Running   0          3m28s
apache-5c9cdffd47-n492h        1/1     Running   0          46s
```
NOTA: Si vemos alguno en Pending, hacer un `kubectl describe <nombre-pod>` y podremos ver un aviso de que el nodo no tiene CPU suficiente para provisionar los pods. El Scheduler no puede crear un pod al no disponer el nodo de suficientes recursos.

