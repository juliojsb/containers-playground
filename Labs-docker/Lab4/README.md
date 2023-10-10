# Docker Compose

En este laboratorio vamos a ver cómo ejecutar aplicaciones mediante Docker Compose.

## 1. Instalar Docker Compose (Standalone, opcional)

NOTA: la versión v2 de compose se incluye como funcionalidad del propio comando docker, por lo que ya no es necesario instalarlo como standalone. No obstante, si quieres utilizar la versión standalone procede con este punto [info](https://docs.docker.com/compose/install/standalone/)

Actualmente la versión con soporte es la V2, ya que la V1 dejó de estar soportada en julio de 2023. Si tenemos instalada la versión 1 como en este ejemplo, tendremos que actualizar:

	$ docker-compose version
	docker-compose version 1.29.2, build 5becea4c
	docker-py version: 5.0.0
	CPython version: 3.7.10
	OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019

Instalamos la versión V2:

	$ sudo curl -SL https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
	$ sudo chmod +x /usr/local/bin/docker-compose
	$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

Comprobamos:

	$ docker-compose --version
	Docker Compose version v2.20.0

## 2. Ejecutar contenedores con Docker Compose

Definimos la aplicación en el fichero compose.yaml. En este caso, utilizaremos como ejemplo un Wordpress y la BBDD MySQL que necesita para ejecutarse:
```
$ vi compose.yaml

services:
  db:
    image: mariadb:10.6.4-focal
    command: '--default-authentication-plugin=mysql_native_password'
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=somewordpress
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=wordpress
  wordpress:
    image: wordpress:latest
    ports:
      - 80:80
    restart: always
    environment:
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=wordpress
      - WORDPRESS_DB_NAME=wordpress
volumes:
  db_data:
```
Levantamos el servicio con `docker-compose up -d`. Este comando se ejecuta en la misma carpeta donde tenemos el fichero YAML

	$ sudo docker-compose up -d
	[+] Running 3/3
	 ✔ Network wordpress_default        Created                                                                                                                                                                                        0.1s 
	 ✔ Container wordpress-db-1         Started                                                                                                                                                                                        0.6s 
	 ✔ Container wordpress-wordpress-1  Started                                                                                                                                                                                        0.6s 

Comprobamos que se han creado los contenedores:

	$ sudo docker ps
	CONTAINER ID   IMAGE                   PORTS                    STATUS          NAMES
	213f59c07dd4   wordpress:latest        0.0.0.0:80->80/tcp       Up 33 seconds   wordpress-wordpress-1
	82e4a77e2134   mariadb:10.6.4-focal    3306/tcp, 33060/tcp      Up 33 seconds   wordpress-db-1

O con:

	$ sudo docker-compose ps

En un navegador, visitamos `http://192.168.122.200`(sustituir por nuestra IP)

Podemos ver el almacenamiento persistente creado (volumen db_data) con el comando `docker volume ls`

	$ sudo docker volume ls
	DRIVER    VOLUME NAME
	local     wordpress_db_data
	
	$ sudo ls /var/lib/docker/volumes/wordpress_db_data/_data
	aria_log.00000001  aria_log_control  ddl_recovery-backup.log  ib_buffer_pool  ibdata1  ib_logfile0  multi-master.info  mysql  performance_schema  sys  wordpress

Paramos y eliminamos los contenedores:
	
	$ sudo docker-compose stop
	[+] Stopping 2/2
	 ✔ Container wordpress-wordpress-1  Stopped                                                                                                                                                                                        1.2s 
	 ✔ Container wordpress-db-1         Stopped                                                                                                                                                                                        0.4s 
	 
	$ sudo docker-compose rm
	? Going to remove wordpress-wordpress-1, wordpress-db-1 Yes
	[+] Removing 2/0
	 ✔ Container wordpress-wordpress-1  Removed                                                                                                                                                                                        0.0s 
	 ✔ Container wordpress-db-1         Removed                                                                                                                                                                                        0.0s 
