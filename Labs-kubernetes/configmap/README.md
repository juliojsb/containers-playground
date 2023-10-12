Vamos a crear un ConfigMap que tiene variables de entorno para la aplicación:

ConfigMap:
```
$ vi myconfigmap.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap
data:
  db_server: "db.example.com"
  database: "mydatabase"
  site.settings: |
    color=blue
    padding:25px
```
Pod:
```
$ vi configmappod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: configmappod
spec:
  containers:
  - name: configmapcontainer
    image: nginx
    env:
      - name: DB_SERVER
        valueFrom:
          configMapKeyRef:
            name: myconfigmap
            key: db_server
      - name: DATABASE
        valueFrom:
          configMapKeyRef:
            name: myconfigmap
            key: database
    volumeMounts:
      - name: config-vol
        mountPath: "/config"
        readOnly: true
  volumes:
    - name: config-vol
      configMap:
        name: myconfigmap
```
Creamos los recursos:
```
$ kubectl apply -f configmap.yaml
$ kubectl apply -f pod.yaml
$ kubectl get pod
```

Comprobamos que se le ha pasado las variables de entorno entrando al pod:
```
kubectl exec -it configmappod -- bash
root@configmappod:/# env | grep -iE 'database\|db_server'
DATABASE=mydatabase
DB_SERVER=db.example.com
```
Si nos fijamos en el yaml del pod, el ConfigMap también se mapea como un volumen (/dev/sda2) y las variables son ficheros dentro de ese volumen. En cada fichero podemos encontrar el contenido de la variable en concreto:
```
root@configmappod:/config# df
Filesystem     1K-blocks     Used Available Use% Mounted on
overlay         50772480 16149824  32011152  34% /
tmpfs              65536        0     65536   0% /dev
tmpfs            5189080        0   5189080   0% /sys/fs/cgroup
/dev/sda2       50772480 16149824  32011152  34% /config
shm                65536        0     65536   0% /dev/shm
tmpfs            6871988       12   6871976   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs            5189080        0   5189080   0% /proc/acpi
tmpfs            5189080        0   5189080   0% /proc/scsi
tmpfs            5189080        0   5189080   0% /sys/firmware
root@configmappod:/config# cd /config/
root@configmappod:/config# ls
database  db_server  site.settings
root@configmappod:/config# cat database
mydatabase
root@configmappod:/config# cat db_server
db.example.com
root@configmappod:/config# cat site.settings
color=blue
padding:25px
```
