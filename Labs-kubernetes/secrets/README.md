# Secrets

Vamos a crear un secret de ejemplo y un pod que utilizará dicho secret. 

En primer lugar codificaremos los valores de las variables en base64:
```bash
$ echo -n "db.example.com" | base64
ZGIuZXhhbXBsZS5jb20=
$ echo -n "admin" | base64
YWRtaW4=
$ echo -n "P@ssw0rd!" | base64
UEBzc3cwcmQh
```
Generamos el secreto (este ejemplo es de tipo Opaque, pero hay [más](https://kubernetes.io/docs/concepts/configuration/secret/#secret-types))
```bash
$ vi secret.yaml
```
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  db_server: ZGIuZXhhbXBsZS5jb20=
  db_username: YWRtaW4=
  db_password: UEBzc3cwcmQh
```
```bash
$ kubectl apply -f secret.yaml
```

Creamos el pod que utilizará el secreto:
```bash
$ vi secretpod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secretenvpod
spec:
  containers:
  - name: secretcontainer
    image: nginx
    env:
      - name: username
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: db_username
      - name: password
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: db_password
      - name: server
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: db_server
    volumeMounts:
    - name: secret-vol
      mountPath: /secret
  volumes:
  - name: secret-vol
    secret:
      secretName: mysecret
```
```bash
kubectl apply -f secretpod.yaml
```

Comprobamos:
```bash
$ kubectl get pod
NAME                           READY   STATUS               RESTARTS      AGE
secretenvpod                   1/1     Running              0             5s
```
Vamos a acceder al pod y comprobar que contiene las variables que le pasamos al secreto:
```bash
$ kubectl exec -it secretenvpod -- bash
root@secretenvpod:/# env | grep -E 'username|password|server'
server=db.example.com
username=admin
password=P@ssw0rd!
root@secretenvpod:/# df
Filesystem                            1K-blocks     Used Available Use% Mounted on
overlay                                36689920 15698944  20990976  43% /
tmpfs                                     65536        0     65536   0% /dev
tmpfs                                   4086788        0   4086788   0% /sys/fs/cgroup
tmpfs                                   8173576       12   8173564   1% /secret
/dev/mapper/vg_data-lv_var_lib_docker  36689920 15698944  20990976  43% /etc/hosts
shm                                       65536        0     65536   0% /dev/shm
tmpfs                                   8173576       12   8173564   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                                   4086788        0   4086788   0% /proc/asound
tmpfs                                   4086788        0   4086788   0% /proc/acpi
tmpfs                                   4086788        0   4086788   0% /proc/scsi
tmpfs                                   4086788        0   4086788   0% /sys/firmware
root@secretenvpod:/# cd /secret/
root@secretenvpod:/secret# ls
db_password  db_server	db_username
```
Como vemos, con acceso a este pod podemos leer información confidencial del secreto. Además. por defecto los secrets no se guardan cifrados en el ETCD, por tanto alguien con permisos suficientes sobre la API de Kubernetes o para editar/leer objetos en el namespace en concreto puede leer estos secretos.

Aunque los secrets están codificados en base64, son fácilmente decodificables:
```bash
$ echo "ZGIuZXhhbXBsZS5jb20=" | base64 -d
db.example.com
```
Conclusión: 
* Siempre debemos proteger el acceso a nuestro clúster siguiendo el principio de mínimo privilegio.
* Se debe cifrar la información confidencial en reposo (rest).
* También podemos hacer uso de Vaults externos que guarden los datos sensibles (AWS, Azure, GCP o [Vault de Hashicorp](https://developer.hashicorp.com/vault/docs/platform/k8s))


