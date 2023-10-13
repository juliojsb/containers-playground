# Certificados

## Certificados de un clúster K8S

Los distintos componentes de un clúster K8S utilizan certificados para comunicarse de forma segura entre ellos:

|Component|Path|
|---|---|
|Kube API Server|/etc/kubernetes/pki/ca.crt|
||/etc/kubernetes/pki/ca.key|
||/etc/kubernetes/pki/apiserver.crt|
||/etc/kubernetes/pki/apiserver.key|
|Kubelet|/var/lib/kubelet/pki/kubelet.crt|
||/var/lib/kubelet/pki/kubelet.key|
||/var/lib/kubelet/pki/kubelet-client-*.pem|
|ETCD|/etc/kubernetes/pki/etcd/ca.crt|
||/etc/kubernetes/pki/etcd/ca.key|
||/etc/kubernetes/pki/etcd/server.crt|
||/etc/kubernetes/pki/etcd/server.key|

## Gestión y renovación de certificados
