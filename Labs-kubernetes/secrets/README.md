
julio@julio-Virtual-Machine:~/curso/secret$ echo -n "db.example.com" | base64
ZGIuZXhhbXBsZS5jb20=

julio@julio-Virtual-Machine:~/curso/secret$ echo -n "admin" | base64
YWRtaW4=

julio@julio-Virtual-Machine:~/curso/secret$ echo -n "P@ssw0rd!" | base64
UEBzc3cwcmQh


[jota@srvdev secret]$ vi secret.yaml

apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  db_server: ZGIuZXhhbXBsZS5jb20=
  db_username: YWRtaW4=
  db_password: UEBzc3cwcmQh


[jota@srvdev secret]$ k apply -f secret.yaml 



[jota@srvdev secret]$ vi secretpod.yaml

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


[jota@srvdev secret]$ k apply -f secretpod.yaml 


[jota@srvdev secret]$ k get pod
NAME                           READY   STATUS               RESTARTS      AGE
cronjob-hello-28184696-4lvft   0/1     ContainerCannotRun   0             7d
cronjob-hello-28184696-4xgtp   0/1     ContainerCannotRun   0             7d
cronjob-hello-28184696-67kfv   0/1     ContainerCannotRun   0             7d
cronjob-hello-28184696-6f92k   0/1     ContainerCannotRun   0             7d
cronjob-hello-28184696-hh9bf   0/1     ContainerCannotRun   0             7d
cronjob-hello-28184696-pvct8   0/1     ContainerCannotRun   0             7d
cronjob-hello-28184696-pxgdz   0/1     ContainerCannotRun   0             7d
cronjob-hello-28194826-hcfps   0/1     Completed            0             2m18s
cronjob-hello-28194827-l5jgb   0/1     Completed            0             78s
cronjob-hello-28194828-bjzpn   0/1     Completed            0             18s
job-hello-1-zgh7x              0/1     Completed            0             7d1h
job-hello-rmjgt                0/1     Completed            0             7d1h
secretenvpod                   1/1     Running              0             5s
web-548f6458b5-ckdwx           1/1     Running              9 (23h ago)   8d
web2-65959ff6d4-kczj2          1/1     Running              9 (23h ago)   8d

[jota@srvdev secret]$ k exec -it secretenvpod -- bash
root@secretenvpod:/# env | grep -E 'username|password|server'
server=db.example.com
username=admin
password=P@ssw0rd!
root@secretenvpod:/# cd /coc^C
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
