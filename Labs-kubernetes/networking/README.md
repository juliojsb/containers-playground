# Networking

Kubernetes implementa un modelo de red en el que tenemos distintas subredes implicadas:​

* Subred de hosts: subred de la que se asignan IPs a los nodos (master, worker...) o a servicios externos.​
* Subred de pods: interna, para los pods del clúster.​
* Subred de servicios: interna, para servicios internos del clúster.​

​En un clúster de Kubernetes existen las siguientes comunicacones:​

* Contenedor a contenedor de un pod: utilizan localhost.​
* Pod a Pod: se pueden comunicar utilizando su IP interna.​
* Pod a Servicio: utilizando la IP de servicio o su DNS asociado.​
* Externas a servicio: utilizando la IP externa del servicio o su DNS asociado.

## Configuración de red del clúster

Podemos ver la configuración del clúster en cuanto a redes de la siguiente manera, utilizando el comando `cluster-info dump`:
```bash
$ kubectl cluster-info dump | grep -m 1 service-cluster-ip-range
                          "--service-cluster-ip-range=10.96.0.0/12",
$ kubectl cluster-info dump | grep -m 1 cluster-cidr
                          "--cluster-cidr=10.244.0.0/16",
```
La información de los rangos de red para pod y servicios también se le pasa al controller manager. Otra forma de comprobarlo por tanto es entrando en un nodo master y comprobando las variables de red que se le pasan al Controller Manager:
```bash
$ minikube ssh
$ docker@minikube:~$ ps -ef  | grep cidr
```
Veremos las variables

*	`cluster-cidr` que es la red de pods
*	`service-cluster-ip-range` para la red de servicios

Ejemplo:
```bash
root        1911    1874  2 10:40 ?        00:00:12 kube-controller-manager --allocate-node-cidrs=true --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf --bind-address=127.0.0.1 --client-ca-file=/var/lib/minikube/certs/ca.crt --cluster-cidr=10.244.0.0/16 --cluster-name=mk --cluster-signing-cert-file=/var/lib/minikube/certs/ca.crt --cluster-signing-key-file=/var/lib/minikube/certs/ca.key --controllers=*,bootstrapsigner,tokencleaner --kubeconfig=/etc/kubernetes/controller-manager.conf --leader-elect=false --requestheader-client-ca-file=/var/lib/minikube/certs/front-proxy-ca.crt --root-ca-file=/var/lib/minikube/certs/ca.crt --service-account-private-key-file=/var/lib/minikube/certs/sa.key --service-cluster-ip-range=10.96.0.0/12 --use-service-account-credentials=true
```
Otra forma es revisando el ConfigMap kubeadm-config
```bash
$ kubectl describe cm kubeadm-config -n kube-system |grep Subnet
```
Veremos:
```bash
podSubnet: 10.244.0.0/16
serviceSubnet: 10.96.0.0/12
```
Para ver las IPs de los nodos:
```bash
$ kubectl get nodes -o wide
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION                CONTAINER-RUNTIME
minikube   Ready    control-plane   46d   v1.27.3   192.168.49.2   <none>        Ubuntu 22.04.2 LTS   3.10.0-1160.81.1.el7.x86_64   docker://24.0.4
```
Para ver las IPs internas de los pods:
```bash
$ kubectl get pod -o wide
NAME                           READY   STATUS               RESTARTS         AGE     IP               NODE       NOMINATED NODE   READINESS GATES
secretenvpod                   1/1     Running              4 (7d17h ago)    31d     10.244.120.112   minikube   <none>           <none>
web-548f6458b5-ckdwx           1/1     Running              13 (7d17h ago)   39d     10.244.120.99    minikube   <none>           <none>
web2-65959ff6d4-kczj2          1/1     Running              13 (7d17h ago)   39d     10.244.120.113   minikube   <none>           <none>
```
Los servicios también tienen una IP, pero de la red de servicios:
```bash
$ kubectl get svc -A
NAMESPACE       NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
default         kubernetes                           ClusterIP   10.96.0.1        <none>        443/TCP                      39d
default         web                                  NodePort    10.110.175.65    <none>        8080:32112/TCP               39d
default         web2                                 NodePort    10.101.105.184   <none>        8080:32533/TCP               39d
ingress-nginx   ingress-nginx-controller             NodePort    10.107.215.140   <none>        80:32065/TCP,443:30994/TCP   39d
ingress-nginx   ingress-nginx-controller-admission   ClusterIP   10.96.102.172    <none>        443/TCP                      39d
kube-system     kube-dns                             ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP       39d
```
## Container Network Interface (CNI)
Existen diversos plugins que se adhieren a este framework para proveer las funcionalidades de red al clúster de Kubernetes: Flannel, Calico, Cilium, WeaveNet...

Las diferencias estriban en soporte de cifrado de comunicaciones, multicast, cómo encapsulan datos (VxLAN o BGP...), soporte oficial corporativo, soporte de Network Policies, etc... Se debe analizar los pros y contras de cada plugin previo a la instalación del clúster de Kubernetes.

Generalmente por facilidad de uso se suele elegir Flannel o WeaveNet, aunque Calico tiene como ventaja el rendimiento...

Podemos encontrar tablas comparativas en:

* https://kubevious.io/blog/post/comparing-kubernetes-container-network-interface-cni-providers#summary-matrix
* https://github.com/weibeld/cni-plugin-comparison#summary-table
