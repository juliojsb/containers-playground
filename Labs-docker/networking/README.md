# Networking
## Drivers

Drivers más utilizados en Docker:

|Tipo|Descripción|
|---|---|
|Bridge|Por defecto. Los contenedores se attachan a una red tipo bridge dentro del host donde se ejecutan​|
|Host|Uso directo de la red del host sin aislamiento.​|
|Overlay|Permite comunicaciones entre distintos procesos de docker y contenedores que se encuentran desplegados en distintos hosts. Es la que se utiliza para Docker Swarm.​|
|None|Contenedores aislados del host y otros contenedores. Útil para procesos batch o que no necesitan acceso de ningún tipo.|

## Redes y Docker-proxy

Podemos listar las redes creadas actualmente con:

	$ sudo docker network ls

Vamos a revisar los contenedores que ahora mismo estamos ejecutando:

	$ docker ps
	CONTAINER ID   IMAGE                   PORTS                    STATUS         NAMES
	03a42266ee8b   registry:2              0.0.0.0:5000->5000/tcp   Up 7 minutes   registry
	16a02d20329a   grafana/grafana         0.0.0.0:3000->3000/tcp   Up 7 minutes   grafana
	8337ee4f0424   influxdb:2.1.1-alpine   0.0.0.0:8086->8086/tcp   Up 7 minutes   influxdb2

¿Cómo se logra acceder a los contenedores desde el propio host o incluso desde fuera? Mediante port forwarding, que realiza el proceso docker-proxy:

	$ ps -ef | grep docker

Veremos entre otros los procesos **docker-proxy** en relación a los contenedores levantados:

	root      1574     1  0 11:25 ?        00:00:01 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
	root      2329  1574  0 11:25 ?        00:00:00 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 5000 -container-ip 172.17.0.2 -container-port 5000
	root      2346  1574  0 11:25 ?        00:00:00 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 8086 -container-ip 172.19.0.2 -container-port 8086
	root      2364  1574  0 11:25 ?        00:00:00 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 3000 -container-ip 172.19.0.3 -container-port 3000

Las peticiones a la host-ip y host-port serán redireccionados al contenedor específico.

## Cambiar subred de docker
