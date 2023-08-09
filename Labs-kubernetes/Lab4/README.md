# Instrucciones Laboratorio 4 - Kubernetes - Almacenamiento Persistente

## Provisionamiento estático

En este laboratorio practicaremos como crear y usar volumenes persistentes estáticos.

1. Abrir una `shell al minikube` y crear un directorio:

       $ minikube ssh
       $ sudo mkdir /mnt/data

2. En el directorio `/mnt/data`, crear un fichero `index.html` y comprobar su contenido:

       $ sudo sh -c "echo 'Hello from Kubernetes storage' > /mnt/data/index.html"
       $ cat /mnt/data/index.html
       Hello from Kubernetes storage

3. Salir de la shell del minikube:

       $ exit

4. Ahora crearemos un volumen persistente de tipo hostPath. Kubernetes admite hostPath para el desarrollo y las pruebas en un clúster de un solo nodo. Un hostPath PersistentVolume utiliza un archivo o directorio en el nodo para emular el almacenamiento conectado a la red. No se debe usar en un clúster en producción. Crearemos el fichero `pv-volume.yaml` con la definición del PV como se indica a continuación:    

       $ vi pv-volume.yaml

       apiVersion: v1
       kind: PersistentVolume
       metadata:
         name: task-pv-volume
         labels:
           type: local
       spec:
         storageClassName: manual
         capacity:
           storage: 3Gi
         accessModes:
           - ReadWriteOnce
         hostPath:
           path: "/mnt/data"

       $ kubectl apply -f pv-volume.yaml

5. Mostrar el PV creado:

       $ kubectl get pv task-pv-volume
       NAME             CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS      CLAIM     STORAGECLASS   REASON    AGE
       task-pv-volume   3Gi       RWO           Retain          Available             manual                   4s

6. El siguiente paso sería crear un PersistentVolumeClaim. Crearemos el fichero `pv-claim.yaml` como se indica a continuación:    

       $ vi pv-claim.yaml

       apiVersion: v1
       kind: PersistentVolumeClaim
       metadata:
         name: task-pv-claim
       spec:
         storageClassName: manual
         accessModes:
           - ReadWriteOnce
         resources:
           requests:
             storage: 3Gi

       $ kubectl apply -f pv-claim.yaml

7. Después de crear el PersistentVolumeClaim, Kubernetes busca un PersistentVolume que satisfaga los requisitos solicitados. Si encuentra un PersistentVolume adecuado se hace el bound del claim y el volúmen. Mostremos otra vez el PV y veremos que en el estado muestre `Bound`:

       $ kubectl get pv task-pv-volume
       NAME             CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS    CLAIM                   STORAGECLASS   REASON    AGE
       task-pv-volume   3Gi       RWO           Retain          Bound     default/task-pv-claim   manual                   2m

8. Ahora veamos el PersistentVolumeClaim:

       $ kubectl get pvc task-pv-claim
       NAME            STATUS    VOLUME           CAPACITY   ACCESSMODES   STORAGECLASS   AGE
       task-pv-claim   Bound     task-pv-volume   3Gi       RWO           manual         30s

9. El siguiente paso sería crear un Pod que use nuestro PVC como se indica a continuación, observe que la referencia en la definición del pod es a un PVC y no ha un PV:

       $ vi pv-pod.yaml

       apiVersion: v1
       kind: Pod
       metadata:
         name: task-pv-pod
       spec:
         volumes:
           - name: task-pv-storage
             persistentVolumeClaim:
               claimName: task-pv-claim
         containers:
           - name: task-pv-container
             image: nginx
             ports:
               - containerPort: 80
                 name: "http-server"
             volumeMounts:
               - mountPath: "/usr/share/nginx/html"
                 name: task-pv-storage

       $ kubectl apply -f pv-pod.yaml

10. Verificar que el pod está corriendo:

        $ kubectl get pod task-pv-pod

11. Ejecute una shell al pod:

        $ kubectl exec -it task-pv-pod -- /bin/bash

12. Desde esta shell del pod verificaremos que nginx esta sirviendo nuestro fichero index.html desde el volumen hostPath (asegurese de estar en la shell del pod):

        # apt update
        # apt install curl
        # curl http://localhost/
        Hello from Kubernetes storage

13. Si vemos el mensaje `Hello from Kubernetes storage` es que todo funciona correctamente.

## Provisionamiento dinámico

En el anterior ejemplo hemos creado un PV y posteriormente un PVC para consumir este almacenamiento. Este ha sido un proceso manual. Vamos a probar a crear un PV automáticamente cuando un PVC lo requiera.

Minikube ya cuenta con una StorageClass que nos permite provisionar almacenamiento de forma dinámica:

       $ kubectl get sc
       NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
       standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  42m

Creamos directamente el Claim:

       $ vi dynamic-pvc.yaml
       
       apiVersion: v1
       kind: PersistentVolumeClaim
       metadata:
         name: dynamic-pvc
       spec:
         accessModes:
         - ReadWriteOnce
         resources:
           requests:
             storage: 1Gi
       
       $ kubectl apply -f dynamic-pvc.yaml

Una vez creado, comprobamos que se ha creado el PVC y tenemos disponsible automáticamente el PV:

       $ kubectl get pvc
       NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
       dynamic-pvc     Bound    pvc-50f025e6-8f87-4a11-81cc-3316a07643c2   1Gi        RWO            standard          3s
       
       $ kubectl get pv
       NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS      REASON   AGE
       pvc-50f025e6-8f87-4a11-81cc-3316a07643c2   1Gi        RWO            Delete           Bound    default/dynamic-pvc     standard                   7s

Si borramos el PVC, se eliminará el PV. Esto es así por la opción de la StorageClass `RECLAIMPOLICY` que está en `Delete`

       $ kubectl delete pvc dynamic-pvc
       
       $ kubectl get pvc
       NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
       
       $ kubectl get pv
       NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS      REASON   AGE




