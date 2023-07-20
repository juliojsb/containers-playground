# Instrucciones Laboratorio 3 - Kubernetes - Controllers
## Deployments

En este laboratorio practicaremos como crear y gestionar Deployments.

1. Crear un namespace llamado `deploy-nx`. Crear el yaml de un `deployment` a partir de la image `nginx:1.7.8`, llamado `nginx`:

       $ kubectl create ns deploy-nx
       $ kubectl create deployment nginx  --image=nginx:1.7.8  --dry-run=client -ndeploy-nx -o yaml > deploy.yaml

2. Editar el fichero yaml del deployment `deploy.yaml`, creado en el paso anterior para que nuestro deploy tenga 2 replicas y definiremos el puerto 80 como el puerto que el contenedor expone:

       $ vi deploy.yaml

 Modificar la cantidad de replicas a 2:

   sustituir:

       replicas: 1

   por:

       replicas: 2

   Modificar la sección **spec.containers** para definir el puerto 80 como containerPort quedando de esta manera:

       spec:
        containers:
        - image: nginx:1.7.8
          name: nginx
          ports:
            - containerPort: 80
          resources: {}

3. Crear el deployment:

       $ kubectl create -f deploy.yaml

4. Mostrar el YAML del deployment:

       $ kubectl get deploy nginx -ndeploy-nx -o yaml

5. Mostrar el `replica set` que creado por este deployment y el YAML:

       $ kubectl get rs -ndeploy-nx
       NAME              DESIRED   CURRENT   READY   AGE
       nginx-5b6f47948   2         2         2       4m34s

       $ kubectl get rs nginx-5b6f47948 -ndeploy-nx -o yaml

6. Comprobar el stado de los rollout del deployment y su histórico:

       $ kubectl rollout status deploy nginx -ndeploy-nx
       deployment "nginx" successfully rolled out

       $ kubectl rollout history deploy nginx -ndeploy-nx
       deployment.apps/nginx
       REVISION  CHANGE-CAUSE
       1         <none>

7. Actualizar la imagen de nginx a la imagen `nginx:1.7.9`:

       $ kubectl set image deploy nginx nginx=nginx:1.7.9 -ndeploy-nx

 Alternativamente se puede hacer editando el recurso, modificando la imagen en el yaml que se nos abre y guardando los cambios:

       $ kubectl edit deploy nginx -ndeploy-nx

8. Comprobar el estado del rollout y el history para confirmar que el rollout funciona correctamente:

       $ kubectl rollout history deploy nginx -ndeploy-nx
       deployment.apps/nginx
       REVISION  CHANGE-CAUSE
       1         <none>

       $ kubectl set image deploy nginx nginx=nginx:1.7.9 -ndeploy-nx
       deployment.apps/nginx image updated

       $ kubectl rollout status deploy nginx -ndeploy-nx
       Waiting for deployment "nginx" rollout to finish: 1 out of 2 new replicas have been updated...
       Waiting for deployment "nginx" rollout to finish: 1 out of 2 new replicas have been updated...
       Waiting for deployment "nginx" rollout to finish: 1 out of 2 new replicas have been updated...
       Waiting for deployment "nginx" rollout to finish: 1 old replicas are pending termination...
       Waiting for deployment "nginx" rollout to finish: 1 old replicas are pending termination...
       deployment "nginx" successfully rolled out

9. Comprobar que se ha creado un nuevo replica set y que este ha creado 2 nuevas replicas del pod y que los nuevos pods tienen configurada la nueva imagen:

       $ kubectl get all -ndeploy-nx
       NAME                         READY   STATUS    RESTARTS   AGE
       pod/nginx-5bf87f5f59-bz5l2   1/1     Running   0          25s
       pod/nginx-5bf87f5f59-p8phf   1/1     Running   0          48s

       NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
       deployment.apps/nginx   2/2     2            2           12m

       NAME                               DESIRED   CURRENT   READY   AGE
       replicaset.apps/nginx-5b6f47948    0         0         0       12m
       replicaset.apps/nginx-5bf87f5f59   2         2         2       48s

       $ kubectl describe po nginx-5bf87f5f59-bz5l2 -ndeploy-nx | grep -i image
         Image:          nginx:1.7.9
         Image ID:       docker-pullable://nginx@sha256:e3456c851a152494c3e4ff5fcc26f240206abac0c9d794affb40e0714846c451
       Normal  Pulled     6m26s      kubelet, minikube  Container image "nginx:1.7.9" already present on machine

10. A continuación vamos a deshacer el rollout y vamos a comprobar como los pods que estan corriendo son del replicaset original (5b6f47948) que la imagen nuevamente es la `nginx:1.7.8`:

        $ kubectl rollout undo deploy nginx -ndeploy-nx

        $ kubectl get all -ndeploy-nx
        NAME                        READY   STATUS    RESTARTS   AGE
        pod/nginx-5b6f47948-4djst   1/1     Running   0          45s
        pod/nginx-5b6f47948-tg4zq   1/1     Running   0          43s

        NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/nginx   2/2     2            2           21m

        NAME                               DESIRED   CURRENT   READY   AGE
        replicaset.apps/nginx-5b6f47948    2         2         2       21m
        replicaset.apps/nginx-5bf87f5f59   0         0         0       9m21s

        $ kubectl describe po nginx-5b6f47948-4djst -ndeploy-nx | grep -i image
            Image:          nginx:1.7.8
            Image ID:       docker-pullable://nginx@sha256:2c390758c6a4660d93467ce5e70e8d08d6e401f748bffba7885ce160ca7e481d
          Normal  Pulled     2m35s      kubelet, minikube  Container image "nginx:1.7.8" already present on machine

11. Lo siguiente que haremos es actualizar el deployment con una imagen incorrecta `nginx:1.91`

        $ kubectl set image deploy nginx nginx=nginx:1.91 -ndeploy-nx

  o editando el deployment, cambiando la imagen en el yaml y guardando los cambios:

        $ kubectl edit deploy nginx -ndeploy-nx

12. Verificar que el nuevo pod no arranca porque da un error con la imagen:

        $ kubectl get po -ndeploy-nx
        $ kubectl describe pod nginx-7789688b8f-m6xst -ndeploy-nx

13. Deshacer este ultimo rollout del deployment indicando esta vez que queremos ir a la revision 2, y verificar que la imagen ahora es `nginx:1.7.9`:

        $ kubectl rollout undo deploy nginx --to-revision=2 -ndeploy-nx
        $ kubectl describe deploy nginx -ndeploy-nx | grep Image:
        $ kubectl rollout status deploy nginx -ndeploy-nx

14. Comprobar los detalles de una revision del history, por ejemplo la 4 que es la que contiene la imagen mala:

        $ kubectl rollout history deploy nginx -ndeploy-nx --revision=4

15. Escalar el deployment a 5 replicas:

        $ kubectl scale deploy nginx --replicas=5 -ndeploy-nx
        $ kubectl get po -ndeploy-nx
        $ kubectl describe deploy nginx -ndeploy-nx

16. Borrar el deployment y comprobar como se borran automaticamente todos los replicasets y los pods:

        $ kubectl delete deploy nginx -ndeploy-nx
        $ kubectl get all -ndeploy-nx
        NAME                         READY   STATUS        RESTARTS   AGE
        pod/nginx-5bf87f5f59-6jzvx   0/1     Terminating   0          6m54s
        pod/nginx-5bf87f5f59-bhg7f   1/1     Terminating   0          2m5s
        pod/nginx-5bf87f5f59-hvwp8   1/1     Terminating   0          2m5s
        pod/nginx-5bf87f5f59-tj78q   0/1     Terminating   0          6m51s
        pod/nginx-5bf87f5f59-wq8g9   0/1     Terminating   0          2m5s

17. Borrar el namespace:

        $ kubectl delete ns deploy-nx


## StatefulSet

En este laboratorio vamos a crear un StatefulSet y comprobaremos cómo se provisiona almacenamiento asociado a cada pod:

1. Creamos el StatefulSet:

       $ vi statefulset.yaml

       apiVersion: apps/v1
       kind: StatefulSet
       metadata:
         name: web
       spec:
         serviceName: "nginx"
         replicas: 2
         selector:
           matchLabels:
             app: nginx
         template:
           metadata:
             labels:
               app: nginx
           spec:
             containers:
             - name: nginx
               image: registry.k8s.io/nginx-slim:0.8
               ports:
               - containerPort: 80
                 name: web
               volumeMounts:
               - name: www
                 mountPath: /usr/share/nginx/html
         volumeClaimTemplates:
         - metadata:
             name: www
           spec:
             accessModes: [ "ReadWriteOnce" ]
             resources:
               requests:
                 storage: 1Gi

       $ kubectl apply -f statefulset.yaml

2. Revisamos los Pods y el StatefulSet

       $ kubectl get pod
       NAME          READY   STATUS    RESTARTS   AGE
       web-0         1/1     Running   0          7s
       web-1         1/1     Running   0          35s

       $ kubectl get sts
       NAME   READY   AGE
       web    2/2     16m

3. Revisamos los PVC. Vemos que se han creado automáticamente y con un nombre identificativo por cada pod:

       $ kubectl get pvc
       NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
       www-web-0       Bound    pvc-d90af041-cbee-4676-a3ba-fb21a5ff1cdb   1Gi        RWO            standard          98s
       www-web-1       Bound    pvc-abff09e2-7478-44bb-b835-7b5edf175a6d   1Gi        RWO            standard          68s

4. Vemos que también se han provisionado los PV:

       $ kubectl get pv
       NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                   STORAGECLASS      REASON   AGE
       pvc-abff09e2-7478-44bb-b835-7b5edf175a6d   1Gi        RWO            Delete           Bound       default/www-web-1       standard                   17s
       pvc-d90af041-cbee-4676-a3ba-fb21a5ff1cdb   1Gi        RWO            Delete           Bound       default/www-web-0       standard                   47s

5. Borramos el StatefulSet:

       $ kubectl delete sts web
       statefulset.apps "web" deleted
   
7. Eliminar un StatefulSet no elimina por defecto el almacenamiento provisionado anteriormente. Si comprobamos los PVC y PV, vemos que no se han borrado. 

## DaemonSet

Para esta parte arrancaremos de forma temporal un clúster de 2 nodos:

	$ minikube stop
	$ minikube start --nodes 2 -p daemonset-demo --driver=docker

Esperamos a que los nodos estén Ready:
	
	$ kubectl get nodes
	NAME                 STATUS   ROLES                  AGE     VERSION
	daemonset-demo       Ready    control-plane,master   6m18s   v1.21.2
	daemonset-demo-m02   Ready    <none>                 116s    v1.21.2

Creamos el DaemonSet:
       
	$ vi daemonset.yaml
	
	apiVersion: apps/v1
	kind: DaemonSet
	metadata:
	 name: fluentd-elasticsearch
	 namespace: kube-system
	 labels:
	   k8s-app: fluentd-logging
	spec:
	 selector:
	   matchLabels:
	     name: fluentd-elasticsearch
	 template:
	   metadata:
	     labels:
	       name: fluentd-elasticsearch
	   spec:
	     tolerations:
	     # these tolerations are to have the daemonset runnable on control plane nodes
	     # remove them if your control plane nodes should not run pods
	     - key: node-role.kubernetes.io/control-plane
	       operator: Exists
	       effect: NoSchedule
	     - key: node-role.kubernetes.io/master
	       operator: Exists
	       effect: NoSchedule
	     containers:
	     - name: fluentd-elasticsearch
	       image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
	       resources:
		 limits:
		   memory: 200Mi
		 requests:
		   cpu: 100m
		   memory: 200Mi
	       volumeMounts:
	       - name: varlog
		 mountPath: /var/log
	     terminationGracePeriodSeconds: 30
	     volumes:
	     - name: varlog
	       hostPath:
		 path: /var/log
	
	$ kubectl apply -f daemonset.yaml

Vemos los dos pods del DaemonSet desplegados en cada nodo. No es necesario especificar el nº de réplicas en el DaemonSet, ya que levantará un pod en cada nodo del clúster.

	$ kubectl get pod -n kube-system -o wide | grep fluentd 
	fluentd-elasticsearch-42hlh              1/1     Running   0          65s     10.244.1.2     daemonset-demo-m02   <none>           <none>
	fluentd-elasticsearch-cq5wf              1/1     Running   0          65s     10.244.0.3     daemonset-demo       <none>           <none>

Modificamos la request del DaemonSet y vemos cómo se actualizan los pods:

	$ kubectl edit ds fluentd-elasticsearch -n kube-system
	
	...
	        resources:
	          limits:
	            memory: 200Mi
	          requests:
	            cpu: 100m
	            memory: 100Mi
	...

El cambio recreará los pods del DaemonSet de forma ordenada:

	$ kubectl get pod -n kube-system -o wide | grep fluentd 
	fluentd-elasticsearch-cq5wf              1/1     Running   0          4m58s   10.244.0.3     daemonset-demo       <none>           <none>
	fluentd-elasticsearch-z9s2x              0/1     Pending   0          0s      <none>         daemonset-demo-m02   <none>           <none>

Reiniciar el DaemonSet con un rollout. Útil si queremos hacer un "refresco" de todos los pods:

	$ kubectl rollout restart ds fluentd-elasticsearch -n kube-system
	daemonset.apps/fluentd-elasticsearch restarted
	
	$ kubectl get pod -n kube-system -o wide | grep fluentd 
	fluentd-elasticsearch-hvqmb              0/1     ContainerCreating   0          1s     <none>         daemonset-demo       <none>           <none>
	fluentd-elasticsearch-z9s2x              1/1     Running             0          5m1s   10.244.1.3     daemonset-demo-m02   <none>           <none>

Observamos cómo se reinician los pods:

	$ kubectl get pod -n kube-system -o wide | grep fluentd 
	fluentd-elasticsearch-hvqmb              1/1     Running       0          5s     10.244.0.5     daemonset-demo       <none>           <none>
	fluentd-elasticsearch-z9s2x              0/1     Terminating   0          5m5s   <none>         daemonset-demo-m02   <none>           <none>

Una vez terminamos las pruebas, paramos el cluster de 2 nodos:

	$ minikube stop -p daemonset-demo
	* Stopping node "daemonset-demo"  ...
	* Powering off "daemonset-demo" via SSH ...
	* Stopping node "daemonset-demo-m02"  ...
	* Powering off "daemonset-demo-m02" via SSH ...
	* 2 nodes stopped.
