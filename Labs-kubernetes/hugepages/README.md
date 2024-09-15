# HugePages

Por defecto, el kernel Linux maneja páginas de memoria de 4K de tamaño. Esto se puede modificar para que sea de 2MB o 1GB.

Óptimo para aplicaciones que hacen gran uso de memoria como ELK, BBDD Oracle, etc...

Esto lo configuramos en el spec del recurso:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: huge-pages-example
spec:
  containers:
  - name: example
    image: fedora:latest
    command:
    - sleep
    - inf
    volumeMounts:
    - mountPath: /hugepages-2Mi
      name: hugepage-2mi
    - mountPath: /hugepages-1Gi
      name: hugepage-1gi
    resources:
      limits:
        hugepages-2Mi: 100Mi
        hugepages-1Gi: 2Gi
        memory: 100Mi
      requests:
        memory: 100Mi
  volumes:
  - name: hugepage-2mi
    emptyDir:
      medium: HugePages-2Mi
  - name: hugepage-1gi
    emptyDir:
      medium: HugePages-1Gi
```

Esto permitiría a este recurso utilizar hasta 100Mi de HugePages de 2Mi y hasta 2Gi de HugePages de 1Gi.

Más [info](https://kubernetes.io/docs/tasks/manage-hugepages/scheduling-hugepages/)
