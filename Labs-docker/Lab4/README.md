# Lab 4: Docker - Compose

En este laboratorio vamos a ver cómo ejecutar aplicaciones mediante Docker Compose.

Definimos la aplicación en el fichero compose.yaml. En este caso, utilizaremos como ejemplo un Wordpress y la BBDD MySQL que necesita para ejecutarse:
  
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

Levantamos el servicio con `docker-compose up -d`. Este comando se ejecuta en la misma carpeta donde tenemos el fichero YAML

	$ docker-compose up -d
	[+] Running 3/3
	 ✔ Network wordpress_default        Created                                                                                                                                                                                        0.1s 
	 ✔ Container wordpress-db-1         Started                                                                                                                                                                                        0.6s 
	 ✔ Container wordpress-wordpress-1  Started                                                                                                                                                                                        0.6s 

Comprobamos que se han creado los contenedores:

	$ docker ps
	CONTAINER ID   IMAGE                   PORTS                    STATUS          NAMES
	213f59c07dd4   wordpress:latest        0.0.0.0:80->80/tcp       Up 33 seconds   wordpress-wordpress-1
	82e4a77e2134   mariadb:10.6.4-focal    3306/tcp, 33060/tcp      Up 33 seconds   wordpress-db-1

En un navegador, visitamos `http://192.168.122.200`(sustituir por nuestra IP)

Podemos ver el almacenamiento persistente creado (volumen db_data) con el comando `docker volume ls`

	$ docker volume ls
	DRIVER    VOLUME NAME
	local     wordpress_db_data
	
	$ sudo ls /var/lib/docker/volumes/wordpress_db_data/_data
	aria_log.00000001  aria_log_control  ddl_recovery-backup.log  ib_buffer_pool  ibdata1  ib_logfile0  multi-master.info  mysql  performance_schema  sys  wordpress