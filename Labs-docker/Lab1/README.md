# Gestión de contenedores

En este laboratorio vamos a poner en practica la gestion de contenedores, teniendo en cuenta que el ciclo de vida de un contenedor puede pasar por estos estados:

|Status|
|---|
|Created|
|Running|
|Paused|
|Stopped|
|Deleted|

## 1. Ejecutar un proceso dentro de un contenedor

Comando para ejecutar un contenedor con un proceso ejecutándose dentro:
```bash
$ sudo docker container run centos ping -c 5 127.0.0.1
```
La salida sería como esta:
```
Unable to find image 'centos:latest' locally
latest: Pulling from library/centos
85432449fd0f: Pull complete
Digest: sha256:3b1a65e9a05...
Status: Downloaded newer image for centos:latest
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.022 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.019 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.029 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.030 ms
64 bytes from 127.0.0.1: icmp_seq=5 ttl=64 time=0.029 ms

--- 127.0.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4103ms
```
En el ejemplo anterior, la imagen del contenedor es `centos` y el proceso que se está ejecutando dentro del contenedor de centos es `ping -c 5 127.0.0.1`, que hace un ping a la dirección de loopback cinco veces hasta que se detiene.

La primera línea es la siguiente:
```
Unable to find image 'centos:latest' locally
```
Esto te indica que Docker no encontró una imagen llamada `centos:latest` en el registro local, así que Docker busca la imagen de algún registro externo. Por defecto, tu entorno Docker está configurado de modo que las imágenes se extraen de Docker Hub en hub.docker.com. Esto se expresa en la segunda línea de la siguiente forma:
```
latest: Pulling from library/centos
```
Las tres líneas de respuesta son las siguientes:
```
85432449fd0f: Pull complete
Digest: sha256:3b1a65e9a05...
Status: Downloaded newer image for centos:latest
```
Estas indican que Docker ha descargado con éxito la imagen, centos:latest, de Docker Hub.

Todas las líneas siguientes de la salida son generadas por el proceso que ejecutaste dentro del contenedor, en este caso es el `ping`.

Es posible que también hayas notado que la palabra clave `latest` aparece algunas veces. Cada imagen tiene una versión (también llamada etiqueta/tag) y si no especifica una versión explícitamente, Docker la asume automáticamente como la última versión (latest).

Si ejecutas el contenedor anterior nuevamente en tu entorno, las primeras cinco líneas de la salida no aparecerán ya que Docker encontrará la imagen del contenedor en el registry local, por lo que no tendrá que descargarla. Inténtalo y verifica:
```bash
$ sudo docker container run centos ping -c 5 127.0.0.1
```
Anteriormente utilizamos la opción `run` que crea el contenedor y a su vez lo pone en estado de ejecución. ¿Si sólo queremos crear el contenedor pero no ejecutarlo de momento? Para ello utilizaremos la opción `create`. Veamos un ejemplo con otra imagen:
```bash
$ sudo docker container create alpine
8c951e5aa1b0309caeaf9093310b5cc6beb0a79b3e0c495b8d6e19833f8e99f5
```
Veremos posteriormente cómo ver el estado de este contenedor.

## 2. Ejecutar un contenedor en background

El comando Docker sería el siguiente:
```bash
$ sudo docker container run -d --name quotes alpine /bin/sh -c "while :; do echo 'esto es una prueba'; printf '\n'; sleep 5; done"
```
En la expresión anterior, usaste dos nuevos parámetros de línea de comando, `-d` y `--name`. La opción `-d` le indica a Docker que ejecute el proceso del contenedor en background. El parámetro `--name` se usa para dar al contenedor un nombre explícito en este ejemplo (quotes).

Si no especificas un nombre de contenedor explícito, Docker asignará automáticamente al contenedor un nombre aleatorio pero único.

Una conclusión importante es que el nombre del contenedor debe ser único. Asegúrate de que el contenedor que acabamos de crear esté activo y en ejecución:
```bash
$ sudo docker container ls -l
```
Comprueba los logs del contenedor:
```bash
$ sudo docker logs <CONTAINER_ID>
```

## 3. Listando contenedores

A medida que continúes ejecutando contenedores a lo largo del tiempo tendrás muchos en tu sistema. Para averiguar qué se está ejecutando actualmente en tu host, puede utilizar el siguiente comando:
```bash
$ sudo docker container ls
```
Esto mostrará una lista de todos los contenedores actualmente en ejecución.

Por defecto, Docker genera siete columnas con los siguientes significados:

![alt Container-ls][imagen1]

[imagen1]: imagenes/Container-ls.png


Si deseas listar todos los contenedores definidos en tu sistema, puede utilizar el parámetro `-a` o `--all` como se muestra a continuación:
```bash
$ sudo docker container ls -a
```
Se mostrará una lista de contenedores en cualquier estado, ya sea created, running o exited (creado, ejecutándose o terminado respectivamente). Veremos el contenedor de alpine creado anteriormente.

A veces, puede ser que solo quieras listar los ID de todos los contenedores. Para lograr esto, tienes que utilizar el parámetro -q:
```bash
$ sudo docker container ls -q
```
Podrías preguntarte cuál sería la utilidad de esto. Aquí hay un ejemplo:
```bash
$ sudo docker container rm -f $(sudo docker container ls -a -q)
```
El comando anterior elimina todos los contenedores actualmente definidos en el sistema, incluidos los detenidos. El comando rm significa remover.


## 4. Detener y ejecutar contenedores

A veces, necesitarás detener temporalmente un contenedor en ejecución. Inicia nuevamente el contenedor del ejemplo anterior:
```bash
$ sudo docker container run -d --name quotes alpine /bin/sh -c "while :; do echo 'esto es una prueba'; printf '\n'; sleep 5; done"
```
Ahora, puedes detener este contenedor con el siguiente comando:
```bash
$ sudo docker container stop quotes
```
Cuando intentes detener el contenedor, probablemente notarás que tarda un tiempo (unos 10 segundos) hasta que se ejecuta. ¿Por qué sucede de esta manera? Docker envía la Señal SIGTERM de linux al proceso principal que se ejecuta dentro del contenedor.

Si el proceso no termina por su cuenta, Docker espera 10 segundos antes de enviar una señal **SIGKILL**, lo que detiene el proceso a la fuerza y finaliza el contenedor.

En el comando anterior, el nombre del contenedor se utiliza para especificar el contenedor específico que se debe detener. Se puede utilizar el ID del contenedor en su lugar.

También podemos pausar un contenedor. Se diferencia del stop en que envia una señal **SIGSTOP**, que "congela" el estado del contenedor sin finalizarlo. Ejecutemos el ejemplo anterior y pongamos en pausa:
```bash
$ sudo docker container run -d --name quotes alpine /bin/sh -c "while :; do echo 'esto es una prueba'; printf '\n'; sleep 5; done"
9e6b2fef52310fa2cd25a46a8ddcadd9e6e6d387935d2e9f1ad21fee8cf424c3
```
Pausamos:
```bash
$ docker pause 9e6b2fef52310fa2cd25a46a8ddcadd9e6e6d387935d2e9f1ad21fee8cf424c3
9e6b2fef52310fa2cd25a46a8ddcadd9e6e6d387935d2e9f1ad21fee8cf424c3
```
Si lo listamos, podemos ver que se ha pausado:
```bash
$ sudo docker container ls -a
CONTAINER ID   IMAGE                                 COMMAND                  CREATED          STATUS                      PORTS                    NAMES
9e6b2fef5231   alpine                                "/bin/sh -c 'while :…"   20 seconds ago   Up 19 seconds (Paused)                               quotes
```
Vamos a dejarlo parado:
```bash
$ sudo docker container stop quotes
```
Y comprobamos:
```bash
$ sudo docker container ls -a
CONTAINER ID   IMAGE                                 COMMAND                  CREATED          STATUS                        PORTS                    NAMES
9e6b2fef5231   alpine                                "/bin/sh -c 'while :…"   3 minutes ago    Exited (137) 20 seconds ago                            quotes
```
## 5. ¿Cómo obtener el ID de un contenedor?

Hay varias maneras de hacerlo. La forma manual es listar todos los contenedores en ejecución y encontrar el que estás buscando en la lista. Sólo tienes que copiar su ID desde allí, por ejemplo:
```bash
$ sudo docker container ls -a
$ sudo docker container ls -a | grep quotes | awk '{print $1}'
```
Aquí utilizamos AWK para obtener el primer campo que es el ID del contenedor.

Con `docker inspect` también podremos ver el ID del contenedor y mucha más información:
```bash
$ docker inspect quotes
```
```json
[
    {
        "Id": "9e6b2fef52310fa2cd25a46a8ddcadd9e6e6d387935d2e9f1ad21fee8cf424c3",
        "Created": "2023-09-30T18:00:43.541075474Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "while :; do echo 'esto es una prueba'; printf '\\n'; sleep 5; done"
        ],
        "State": {
            "Status": "exited",
            "Running": false,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 0,
            "ExitCode": 137,
            "Error": "",
            "StartedAt": "2023-09-30T18:00:43.84942706Z",
            "FinishedAt": "2023-09-30T18:03:24.817199539Z"
```

## 6. Cómo limitar el uso de CPU y memoria

Por defecto, los contenedores pueden llegar a utilizar toda la CPU y memoria del host donde se ejecutan si lo precisan. Esto puede afectar a la disponibilidad de recursos y rendimiento de otros procesos que se ejecutan en el sistema. Docker permite establecer límites al uso de recursos en cada contenedor.

Ejecutamos un contenedor sin especificar límites:
```bash
$ sudo docker container run --name mynginx1 -p 80:80 -d nginx
8bcbe22323a1ea77b97a53408f1859f415b323f827c58d3ac77c2ccbc081eb45

$ sudo docker ps
CONTAINER ID   IMAGE                   PORTS                    STATUS          NAMES
8bcbe22323a1   nginx                   0.0.0.0:80->80/tcp       Up 3 seconds    mynginx1
```
Con el comando `docker stats` podemos ver el uso de recursos de nuestros contenedores:
```bash
$ sudo docker stats --no-stream
CONTAINER ID   NAME               CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O        PIDS
8bcbe22323a1   mynginx1           0.00%     3.797MiB / 7.795GiB   0.05%     0B / 0B           27MB / 21.5kB    3
```
Como vemos, el contenedor no tiene límites en el uso de CPU/memoria. Paramos y eliminamos el contenedor:
```bash
$ sudo docker stop 8bcbe22323a1
$ sudo docker rm 8bcbe22323a1
```
Lo ejecutamos especificando que puede llegar a utilizar 1GB de memoria RAM y 0.5 CPUs:
```bash
$ sudo docker container run --name mynginx1 -m 1024MB --cpus ".5" -p 80:80 -d nginx
0be41e6b37d85448a4d374a80214cf7c2d17c966025db63fc18c7f9513033902
```
Comprobamos con docker stats que se ha establecido el límite:
```bash
$ sudo docker stats --no-stream
CONTAINER ID   NAME               CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O        PIDS
0be41e6b37d8   mynginx1           0.00%     2.363MiB / 1GiB       0.23%     0B / 0B           0B / 5.12kB      3
```
Si inspeccionamos el contenedor, también lo vemos:
```bash
$ sudo docker inspect 0be41e6b37d8
...
            "Memory": 1073741824,
            "NanoCpus": 500000000,
...
```
También hay limits para GPU en caso de aplicaciones que hagan uso de procesamiento gráfico.

## 7. Eliminando contenedores

El comando para eliminar un contenedor es el siguiente:
```bash
$ sudo docker container rm <container ID o container name>
```
Para nuestro contenedor por ejemplo:
```bash
$ sudo docker container rm quotes
```
A veces, eliminar un contenedor en ejecución no funcionará; Si desea forzar la eliminación, puedes utilizar el parámetro de línea de comandos `-f` o `--force`.

IMPORTANTE: Eliminar un contenedor no elimina la imagen del mismo. Si hacemos `docker image ls` después de parar el contenedor, veremos la imagen.
```
REPOSITORY                                    TAG               IMAGE ID       CREATED         SIZE
mycron                                        latest            879a54e8017e   11 months ago   32.9MB
debian                                        bullseye-slim     6a8065e4ba13   11 months ago   80.4MB
alpine                                        3.14              5977be310a9d   12 months ago   5.59MB
alpine                                        3                 d7d3d98c851f   12 months ago   5.53MB
registry                                      2                 773dbf02e42e   13 months ago   24.1MB
registry.redhat.io/rhel7/rhel                 latest            acf3e09a39c9   14 months ago   206MB
registry.redhat.io/openjdk/openjdk-11-rhel7   latest            9bcac2eabc8b   14 months ago   530MB
```
