# Almacenamiento persistente

En este laboratorio vamos a ver cómo podemos persistir datos después de borrar un contenedor de dos formas:

* Volumes
* Bind Mounts

## Volumes

Ejecutamos un contenedor de Nginx:

    $ sudo docker container run --name mynginx1 -p 80:80 -d nginx

Comprobamos que se está ejecutando:

    $ docker ps
    CONTAINER ID   IMAGE                   PORTS                    STATUS         NAMES
    6b7b521d50a2   nginx                   0.0.0.0:80->80/tcp       Up 3 seconds   mynginx1

Entramos al contenedor y creamos un fichero de prueba:

    $ docker exec -it mynginx1 bash
    
    root@6b7b521d50a2:/# 
    
    root@6b7b521d50a2:/# env
    HOSTNAME=6b7b521d50a2
    PWD=/
    PKG_RELEASE=1~bookworm
    HOME=/root
    NJS_VERSION=0.7.12
    TERM=xterm
    SHLVL=1
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    NGINX_VERSION=1.25.1
    _=/usr/bin/env
    
    root@6b7b521d50a2:/# echo "Esto es un fichero de prueba" > /tmp/readme.txt
    
    root@6b7b521d50a2:/# cat /tmp/readme.txt
    Esto es un fichero de prueba
    
    root@6b7b521d50a2:/# exit
    exit

Paramos y eliminamos el contenedor:

    $ sudo docker container stop mynginx1
    mynginx1
    
    $ sudo docker container rm mynginx1
    mynginx1

Volvemos a ejecutarlo y entramos. Comprobamos que el fichero que creamos anteriormente no existe. Esto es porque el contenido del contenedor es efímero:
    
    $ docker exec -it mynginx1 bash
    
    root@23164c93723e:/# cat /tmp/readme.txt
    cat: /tmp/readme.txt: No such file or directory

Lo ejecutamos creando un volumen y montándolo en el contenedor:

    $ docker container run --name mynginx1 -p 80:80 -d --mount source=myvol,target=/tmp  nginx
    a949bce0d85b2f65d42eb7a02caf20a9b81dd6c0a8f6f3de2772f80fab43cd78

Podemos comprobar el volumen creado:

	$ docker volume ls

Y obtener algo más de detalle con `inspect`:

	$ docker volume inspect myvol
	[
	    {
	        "CreatedAt": "2023-09-25T19:15:26+02:00",
	        "Driver": "local",
	        "Labels": null,
	        "Mountpoint": "/var/lib/docker/volumes/myvol/_data",
	        "Name": "myvol",
	        "Options": null,
	        "Scope": "local"
	    }
	]

Entramos al contenedor y creamos un fichero con datos:

    $ docker exec -it mynginx1 bash

    root@a949bce0d85b:/# echo "Esto es un fichero de prueba" > /tmp/readme.txt
    
    root@a949bce0d85b:/# cat /tmp/readme.txt
    Esto es un fichero de prueba

Podemos comprobar con `df` dentro del contenedor cómo el volumen está montado:

    root@a949bce0d85b:/# df
    Filesystem                            1K-blocks     Used Available Use% Mounted on
    overlay                                36689920 19454764  17235156  54% /
    tmpfs                                     65536        0     65536   0% /dev
    tmpfs                                   4086788        0   4086788   0% /sys/fs/cgroup
    shm                                       65536        0     65536   0% /dev/shm
    /dev/mapper/vg_data-lv_var_lib_docker  36689920 19454764  17235156  54% /tmp
    tmpfs                                   4086788        0   4086788   0% /proc/asound
    tmpfs                                   4086788        0   4086788   0% /proc/acpi
    tmpfs                                   4086788        0   4086788   0% /proc/scsi
    tmpfs                                   4086788        0   4086788   0% /sys/firmware

Salimos, lo paramos y borramos:

    $ sudo docker container stop mynginx1
    mynginx1
    
    $ sudo docker container rm mynginx1
    mynginx1

Lo creamos de nuevo, entramos y comprobamos que el fichero sigue existiendo:

	$ docker container run --name mynginx1 -p 80:80 -d --mount source=myvol,target=/tmp  nginx
	011b94a078066af6c3b74ac8af0d15a3f1566604e776dc36852eec9454d19ec6
    
	$ docker exec -it mynginx1 bash
	root@011b94a07806:/# cat /tmp/readme.txt 
	Esto es un fichero de prueba

De esta manera, podemos persistir los datos incluso cuando hemos borrado el contenedor.

### Cambiar punto de montaje de volúmenes docker
Los volúmenes docker por defecto se crean dentro de la estructura `/var/lib/docker`. Si queremos cambiar este path por defecto:

1. Preparamos el nuevo path `/new/path/docker`, que puede ser un directorio sin más o un FS independiente separado del principal (recomendado)

2. Editamos el servicio de docker:
```
vi /lib/systemd/system/docker.service
```
En la línea:
```
ExecStart=/usr/bin/dockerd -H fd://
```
Añadimos la opción -g con el path nuevo:
```
ExecStart=/usr/bin/dockerd -g /new/path/docker -H fd://
```
Recargamos y reiniciamos el servicio:
```
systemctl daemon-reload
systemctl restart docker
```
## Bind Mounts

Vamos a crear almacenamiento persistente con Bind Mounts:

	$ mkdir -p /tmp/data/mynginx1
	$ docker container run --name mynginx1 -p 80:80 -d --mount type=bind,source=/tmp/data/mynginx1,target=/tmp  nginx

Si comprobamos los volúmenes de docker, no lo veremos listado:

	$ docker volume ls

Este volumen ya no es gestionado internamente por docker. Es útil si queremos tener separado este volumen y hacemos LVM snapshots, mirroring... por fuera de las funcionalidades que proporciona docker.
