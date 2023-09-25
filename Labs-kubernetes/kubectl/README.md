# Kubectl

## Instalación
```shell
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
## Kubeconfig Tips
- Ver toda la configuración de kubeconfig
```shell
kubectl config view # Show Merged kubeconfig settings.
```

- Utilizar multiples kubeconfig al mismo tiempo en una vista conjunta
```shell
KUBECONFIG=~/.kube/config:~/.kube/kubconfig2 
kubectl config view
```

- Obtener password para e2e user
```shell
kubectl config view -o jsonpath='{.users[?(@.name == "e2e")].user.password}'
kubectl config view -o jsonpath='{.users[].name}'    # display the first user
kubectl config view -o jsonpath='{.users[*].name}'   # get a list of users
kubectl config get-contexts                          # display list of contexts 
kubectl config current-context                       # display the current-context
kubectl config use-context my-cluster-name           # set the default context to my-cluster-name
```

- Añadir un nuevo kubeconf que soporta basic auth
```shell
kubectl config set-credentials kubeuser/foo.kubernetes.com --username=kubeuser --password=kubepassword
```
- Utilizar un namespace especifico para todos los siguientes comandos kubectl en el contexto.
```shell
kubectl config set-context --current --namespace=ggckad-s2
kubectl config set-context gce --user=cluster-admin --namespace=foo && kubectl config use-context gce # Specific username and namespace
kubectl config unset users.foo                       # delete user foo
```

- Acceso a la API de Kubernetes
```shell
APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
TOKEN=$(kubectl get secret $(kubectl get serviceaccount default -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.token}' | base64 --decode )
curl $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure
```

## Querys

- Query a la API de Prometheus para obtener el porcentaje de ocupación de los PVs
```shell
curl -ks -X GET -H 'Authorization: Bearer $TOKEN' "https://prometheus-k8s-openshift-monitoring.apps.cluster.ocp.local/api/v1/query?query=(kubelet_volume_stats_used_bytes*100)/kubelet_volume_stats_capacity_bytes" | jq  -r '.data.result[] |"\(.metric.persistentvolumeclaim)=\(.value[1])%"'
```

- Para saber que pods usan una imagen determinada:
```shell
kubectl get pods -o json | jq -r '.items[] | select(.metadata.name | test("test-")).spec.containers[].image'
```

## Productividad
### Alias
Añadir a nuestro `.bashrc` o fichero de configuración de shell predeterminado alias útiles para Kubernetes:

	alias k='kubectl'
	alias kg='kubectl get'
	alias kgpo='kubectl get pod'

Para más alias ver: 

	https://github.com/ahmetb/kubectl-aliases

### Autocompletado

Previamente instalamos el paquete correspondiente según nuestra distribución Linux:

	apt-get install bash-completion 
	yum install bash-completion

Añadimos a nuestro `.bashrc`

	echo 'source <(kubectl completion bash)' >>~/.bashrc

Si utilizamos alias, adicionalmente:

	echo 'alias k=kubectl' >>~/.bashrc
	echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
 
### Plugins
Los plugins extienden las funcionalidades de kubectl.

Una forma sencilla de instalarlos es mediante krew. IMPORTANTE: no se audita la seguridad de los plugins que se instalan adicionalmente

1.Instalamos krew:

	https://krew.sigs.k8s.io/docs/user-guide/setup/install/

2.Instalamos plugins esenciales:

	kubectl krew install ctx ns resource-capacity who-can

Docs:
	
	https://github.com/ahmetb/kubectx
	https://github.com/robscott/kube-capacity
 	https://github.com/aquasecurity/kubectl-who-can
  	https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

