# Instrucciones Laboratorio 2 Docker - Manejo de Imagenes

En este laboratorio vamos a crear una imagen de las tres formas que hemos comentado, de forma interactiva, a partir de un docker file o desde un fichero comprimido creado a partir de una imagen existente. 
La aplicación es una API de `nodejs` que da la hora actual. Para ellos usaremos `express lib`. Veamos cómo se ve desde la perspectiva de las imágenes de docker.

![alt Imagenes][imagen]

[imagen]: imagenes/Imagenes.png


## 1. Creando la imagen de forma interactiva (modificando el contenedor)

Cada imagen parte de una imagen base, así que lo primero que haremos será hacer el pull de la imagen que queremos que sea nuestra imagen base:

    $ docker pull node:latest

Ahora ya nos hemos traido la imagen base. Para añadirle capas de forma interactiva, arrancaremos un contenedor usando dicha imagen, entraremos en el y ejecutaremos todo lo que necesitamos para crear nuestra imagen personalizada.

Arrancamos el contenedor:

    $ sudo docker container run -dit --name node-express-server node

Entramos en el contenedor:

    $ sudo docker exec -it node-express-server /bin/bash

Una vez dentro del contenedor crearemos directorios y ficheros y finalmente con un `docker commit` crearemos una nueva imagen ya personalizada.

Dentro del contenedor creamos un directorio:

    $ mkdir /usr/src/app

Nos movemos al directorio:

    $ cd /usr/src/app

Creamos el package.json (confirmar todas las respuestas por defecto):

    $ npm init

Instalar el express:

    $ npm install --save express

Instalamos el vim para usarlo como editor:

    $ apt-get update
    $ apt-get install -y vim

Creamos un fichero index.js:

    $ cat > index.js <<EOF
    const express = require('express');
    const app = express();
    const port = 3080;
    
    app.get('/time', (req,res) => {
        res.send(new Date());
    });
    
    app.get('/', (req,res) => {
        res.send('App Works !!!!');
    });
    
    app.listen(port, ()=> {
        console.log(\`server listening on the port:::\${port}\`);
    });
    EOF

Comprobar si el contenido del fichero index.js es correcto:

    $ cat index.js

Ahora probamos la aplicación ejecutando el index.js:

    $ node index.js
    server listening on the port:::3080

    // ctrl c y exit para salir
    exit

En este punto la imagen esta personalizada tal y como la queremos, ya podemos crear una nueva imagen llamada `my-node-server` a partir de las modificaciones que hemos hecho al contenedor:

    $ sudo docker container commit node-express-server my-node-server

Comprobamos que la imagen se crea correctamente:

    $ sudo docker images
    REPOSITORY                                              TAG                  IMAGE ID            CREATED             SIZE
    my-node-server                                          latest               8e21de81395b        20 seconds ago      994MB

A continuación arrancaremos un contenedor de nombre `current-time` con la imagen que hemos creado:

    $ sudo docker container run -dit --name current-time -p 3080:3080 my-node-server node /usr/src/app/index.js

Para probar la imagen abrir en un navegador `localhost:3080/time`


## 2. Creando la imagen usando un Dockerfile

Veamos ahora cómo podemos construir la misma imagen a partir de un `dockerfile`. Dockerfile se usa para automatizar el proceso de creación de imágenes. Una vez que tengamos un Dockerfile, se pueden crear tantas imágenes como sea desee.

En el repositorio de la formación esta el `Dockerfile` que usaremos para crear la imagen en el directorio `node-express-server`. En este directorio tambien estan los ficheros `package.json` e `index.js` que se necesitan para la imagen. Nos movemos al directorio `node-express-server`:

    $ cd <repo>/kubernetes/Labs-docker/Lab2/node-express-server

Este es el contenido del `Dockerfile`, los pasos que se describen son los mismos que hicimos manualmente en el punto 1:

    # base image
    FROM node:latest

    # setting the work direcotry
    WORKDIR /usr/src/app

    # copy package.json
    COPY ./package.json .

    # install dependencies
    RUN npm install

    # COPY index.js
    COPY ./index.js .

    RUN node -v

    CMD ["node","index.js"]

Para crear la imagen a partir del Dockerfile, ejecutaremos un `docker build`:

    $ sudo docker build -t my-node-server:1.0 .

En este punto tendremos en local la nueva imagen:

    $ sudo docker images
    REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
    my-node-server                1.0                 69fb5d3d952c        2 minutes ago       940MB

 y podriamos arrancar un contenedor a partir de dicha imagen:

    $ sudo docker container run -dit --name new-node-server my-node-server:1.0

Si entramos en el contenedor que acabamos de crear veremos que el directorio `/usr/src/app` contiene los ficheros que se indicaba que se copiaran en el Dockerfile:

    $ sudo docker exec -it new-node-server /bin/bash
    root@4142e9490d90:/usr/src/app# ls -la
        total 60
    drwxr-xr-x  1 root root  4096 Dec  3 18:01 .
    drwxr-xr-x  1 root root  4096 Dec  3 18:01 ..
    -rw-rw-r--  1 root root   292 Dec  3 17:19 index.js
    drwxr-xr-x 59 root root  4096 Dec  3 18:01 node_modules
    -rw-r--r--  1 root root 39693 Dec  3 18:01 package-lock.json
    -rw-rw-r--  1 root root   555 Dec  3 17:19 package.json


Salir del contenedor con `exit`

## 3. Creando la imagen importando un fichero comprimido

Con este método podemos guardar una imagen que tengamos en un fichero `.tar` (con el comando `docker save`) y luego podemos importar la misma imagen que tenemor en el fichero `.tar` donde lo necesitemos (con el comando `docker load`).

Vamos a guardar la imagen que creamos en el caso anterior (my-node-server:1.0) en un fichero .tar:

    $ sudo docker save my-node-server:1.0 > mynodeserver.tar

A continuación vamos a borrar la imagen (si el contenedor aun esta corriendo hay que pararlo y borrarlo antes de borrar la imagen):

    $ sudo docker stop new-node-server
    new-node-server
    $ sudo docker rm new-node-server
    new-node-server
    
    $ sudo docker image ls |grep my-node-server
    my-node-server                                          1.0                  4c9d7b781a89        8 minutes ago       945MB
    my-node-server                                          latest               8e21de81395b        21 hours ago        994MB

    $ sudo docker image rm my-node-server:1.0

Y ahora vamos a volver a crear la imagen a partir del fichero tar:

    $ sudo docker load < mynodeserver.tar
    bf5297e6e7ac: Loading layer [==================================================>]  3.072kB/3.072kB
    3c6a1dc4cadc: Loading layer [==================================================>]  4.096kB/4.096kB
    0175ea967955: Loading layer [==================================================>]  4.029MB/4.029MB
    955b95942808: Loading layer [==================================================>]  3.584kB/3.584kB
    Loaded image: my-node-server:1.0

A partir de aquí se podría arrancar un contenedor usando la imagen que hemos cargado. Esta opción es muy útil si se quiere compartir imagenes.

IMPORTANTE: No confundir guardar una imagen con `docker save` con hacer backup de contenedores. Simplemente estamos guardando la imagen, no el contenido de ningún contenedor.

## 4. Limpieza de imágenes
Todas las imágenes, builds, etc... se guardan en disco para ser reutilizados en caso necesario. Si necesitamos liberar espacio u optimizar su uso eliminando recursos que ya no necesitemos, podemos hacer limpieza de las imágenes no utilizadas por ningún contenedor o que no tenga tag asociado con:

    $ docker image prune

Existen comandos para limpiar recursos como networks, builds, etc... no utilizados:

    $ docker container prune
    $ docker network prune
    $ docker image prune
    $ docker builder prune

Y un comando para limpiar todo a la vez:

    $ docker system prune

Ejemplo:

    $ docker system prune
    WARNING! This will remove:
      - all stopped containers
      - all networks not used by at least one container
      - all dangling images
      - all dangling build cache
    
    Are you sure you want to continue? [y/N] y
    Deleted Containers:
    973970093fcba95faa454bda8a3dee065f4f482b85e8a9e7667de4636892e799
    37999ac56dfadb31863cd2d3a07492c245311561ebe66e4b00d9249fe731b65d
    97ec213e36cfac9c0c9cfcca9078608a0bba549707b2f07b1badf39bb2dbf8bc
    1a8193156c56e081ca579473a04aec3d84b2d7bca6860091863d9de5b99eea1d
    5da586414d7dc13d0b160301341c637964a29c069860026d3b83be0008667cfb
    a5454d80637c829f1540ffce4911da4d2b31f9deb217341a5deeabb34b415975
    4e05681f0a449a5d9338af227f326c0ee042be16cd7b413c083aa1c9457d8da7
    
    Deleted Networks:
    daemonset-demo
    minikube
    
    Total reclaimed space: 11.7MB
