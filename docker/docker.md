# Curso de Docker, Podman y Kubernetes

## Conceptos Básicos de Docker
    $ docker images
    $ docker run hello-world
    - Eliminar todas las imagenes con PowerShell
        $ docker rmi $(docker images -q)
    - Elimina las imagenes colgantes
        $ docker rmi $(docker images -f "dangling=true" -q)
        $ docker image prune
    - Elimina todos los contenedores
        $ docker rm $(docker ps -q -a)
    - Especificar nombre al contenedor
        $ docker run --name HolaMundo hello-world
    - Parar contenedores
        $ docker stop HolaMundo 
        $ docker kill HolaMundo
    - Iniciar un contenedor parado
        $ docker start HolaMundo
    
### Comandos básicos
Comando	        | Acción
----------------|----------------------------------------------------------
docker ps	    | Lista los contenedores ejecutándose
docker ps -a	| Lista todos los contenedores
docker run	    | Crea un contenedor a partir de una imagen
docker images	| Lista las imágenes de docker descargadas en mi ordenador
docker start	| Reinicializa un contenedor parado
docker stop	    | Para (correctamente) un contenedor en marcha
docker kill	    | Mata un contenedor en marcha
docker rmi	    | Elimina una imagen de mi ordenador


### Gestión de imagenes
> pull para descargar imágenes de un repositorio
> images para listar las imágenes que tengas descargadas
> rmi para borrar imágenes


### Gestión de puertos
    Exponemos un puerto para poder interactuar con el contenedor
    $ docker run -d -p puerto_host:puerto_contenedor dockercampusmvp/pingpong
    Buscar una IP de un contenedor:
    $ docker inspect <id-contenedor> -f "{{ .NetworkSettings.IPAddress }}"
    Ejecutar contenedor en segundo plano
    $ docker run -d test

### Gestión de volumenes
    Se gestionan por Docker, es para persistir ficheros entre contenedores.
    Se pueden crear volumenes propios para compartir entre contenedores
    $ docker volumen create <my-volumen>
    
    Utilizar ese volumen con un contenedor
    $ docker run -d -v my-volumen:/test-volumen test-volumen

    Visualizar los volumenes creados
    $ docker volumen ls

    Eliminar un volumen
    $ docker volumen rm <my-volumen>

    Eliminar todos los volumenes 
    $ docker volumen prune

    Se pueden utilizar Bind Mounts, que son volumenes que referencian a la máquina Host para compartir ficheros con los contenedores.
    Un ejemplo sería montar un servidor SFTP y compartir ahí los archivos.
    docker run -d -v c:\jlparres\Docker\volumen:/test-volumen test-vollumen

### Docker System
    Espacio ocupado en disco
        $ docker system df
        $ docker system df -v
    Limpieza de espacio ocupado
        $ docker system prune
    

### Examen Parte 1
    -> Marca todas las afirmaciones correctas sobre los comandos CMD y ENTRYPOINT
        CMD admite dos sintaxis (shell y exec)
        Flecha que indica la opción elegida ENTRYPOINT admite dos sintaxis (shell y exec)
        Flecha que indica la opción elegida Si se usa ENTRYPOINT y CMD a la vez, CMD es ignorado si ENTRYPOINT se usa en modo shell

    -> De las siguientes situaciones, ¿cuál te obliga a usar el modificado -f en "docker rmi " para borrar una imagen?
        Si hay algún contenedor en marcha de dicha imagen -f lo para y borra la imagen

    -> ¿Qué diferencias hay entre bind mounts y volúmenes?
        Los volúmenes pueden persistir en almacenamientos externos. Los bind mounts solo en el host
        Los volúmenes son manejados por Docker, los bind mounts solo mapean una carpeta del host al contenedor
        La CLI docker volumes permite ver y eliminar volúmenes, pero no bind mounts

    -> Dadas esas tres casuísticas:
        Compartir ficheros de configuración con un contenedor
        Permitir al contenedor acceder a datos no persistentes pero de alto rendimiento
        Ofrecer al contenedor un espacio de almacenamiento persistente
        ¿Qué usarías para cada casuística? (Marca la mejor opción)
            1- bind mount, 2- tmpfs, 3- volumen

### Buenas prácticas para crear un Dockerfile
    https://www.campusmvp.es/recursos/post/mejores-practicas-para-crear-dockerfiles-excelentes.aspx


## Escenarios Multicontenedor - Docker Compose
    - Bind Mounts -> carpeta de ficheros alojada en el host
        La idea es que la configuración no está en la imagen, sino que sea proporcionada por el entorno. 
        La configuración forma parte del entorno y no de la imagen. 
        Usar un bind mount es la manera de que el entorno (el host) proporcione ficheros de configuración al contenedor.
    - Variables de entorno
        La configuración forma parte del propio entorno.
        El modificador -e nos permite establecer variables de entorno
            $ docker run -e "variable=valor" <nombre_imagen>
    - Vamos a utilizar una configuración alojada en el propio host
        $ docker run -d -p 5000:80 dockercampusmvp/cfgweb
        $ docker run -d -p5001:80 -v /C/jlparres/Docker/bindmount:/app/cfg dockercampusmvp/cfgweb
    - Utilizando variables de entorno
        $ docker run -d -p5002:80 -v /C/jlparres/Docker/bindmount/empty:/app/cfg -e "ConnectionStrings__DefaultConnection=prod.database" dockercampusmvp/cfgweb

    - Levantando contenedores desde un compose especifico
        $ docker compose -f docker-compose.dev.yml up -d
        $ docker compose -f docker-compose.prod.yml up -d
    - Levantando contenedores por entornos y distintos ficheros
        $ docker compose -f docker-compose.yml -f docker-compose.prod.yml up
    - Revisar el yaml generado por varios ficheros
        $ docker compose -f docker-compose.yml -f docker-compose.prod.yml config

    - Perfiles
        Levanta los servicios frontend y db en el contenedor
            $ docker compose --profile front up
        Levanta los servicios adminer y db
            $ docker compose --profile debug up
        Levanta solo el servicio de db porque no está asociado a ningún perfil
            $ docker compose --profile debug up

    - Creando imagenes
        - No es necesario el comando $ docker build
            $ docker compose -f .\docker-compose-build.yaml build

### Red interna de docker  
    $ docker run -d -p3000:3000 dockercampusmvp/nodejs-sample
    $ docker run -d -e "PORT=80" dockercampusmvp/nodejs-sample
    
    - Buscar la ip del contenedor 
        $ docker inspect -f "{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}" <id-contenedor>
    - Hacer ping a través del contenedor
        $ curl http://localhost:3000/get?url=http://172.17.0.4/ping

    > Cuando accedas a un contenedor desde tu máquina, la IP de este contenedor es localhost (ya que el contenedor se ejecuta en tu máquina).
    > Cuando accedas a un contenedor desde tu máquina, dicho contenedor debe tener un puerto mapeado y debes usar el puerto del host (de tu máquina), no el puerto real del contenedor.
    > Cuando accedas a un contenedor desde otro contenedor debes usar la IP interna del otro contenedor.
    > Cuando accedas a un contenedor desde otro contenedor debes usar el puerto real del otro contenedor. El puerto del host mapeado es irrelevante.
    > Desde dentro de un contenedor, localhost es el propio contenedor.

### Ejecutar sesiones interactivas
    Abrir una sesión interactiva con el contenedor
        $ docker run -it dockercampusmvp/echo

    Crear Docker compose para habilitar en varios contenedores abrir sesiones intectivas con los contenedores.
    Esto quiere decir, que podemos interactuar con el contenedor
        # docker-compose.yaml
            services:
            echo1:
                image: dockercampusmvp/echo
                environment:
                - color=Red
                stdin_open: true # Indica que puedes escribir comandos (entrada de datos strdin)
                tty: true        # Indica que tiene salida (salida de datos tty)
            echo2:
                image: dockercampusmvp/echo
                environment:
                - color=Green

### Escalado de instancias 
    - Utilizamos el proyecto fixedguid
        # docker-compose.yaml
            services:
            fixedguid:
                image: dockercampusmvp/fixedguid
                ports:
                - "5000:80"

    - Vamos a escalar las instancias 
        $ docker compose up --scale fixedguid=5
            Creamos 5 instancias, pero como estamos mapeando el mismo puerto, nos dará error
    
    - Escalando instancias sin mapear puerto
        Indicaremos el puerto que expone el contenedor y el propio host, será quien mapee el puerto del host
            # docker-compose.yaml
            services:
                fixedguid:
                    image: dockercampusmvp/fixedguid
                    ports:
                        - 80

        $ docker compose up -d --scale fixedguid=5
            PS C:\jlparres\GIT\Docker\02-Multicontenedores\escalado> docker ps
            CONTAINER ID   IMAGE                       COMMAND                  CREATED         STATUS         PORTS                   NAMES
            9c7276713d66   dockercampusmvp/fixedguid   "dotnet FixedGuid.dll"   5 seconds ago   Up 3 seconds   0.0.0.0:61789->80/tcp   escalado-fixedguid-4
            dad9f7a338ef   dockercampusmvp/fixedguid   "dotnet FixedGuid.dll"   5 seconds ago   Up 3 seconds   0.0.0.0:61791->80/tcp   escalado-fixedguid-5
            dedd88fd73d3   dockercampusmvp/fixedguid   "dotnet FixedGuid.dll"   5 seconds ago   Up 3 seconds   0.0.0.0:61793->80/tcp   escalado-fixedguid-2
            103f75ccad07   dockercampusmvp/fixedguid   "dotnet FixedGuid.dll"   5 seconds ago   Up 2 seconds   0.0.0.0:61794->80/tcp   escalado-fixedguid-3
            313c666b6210   dockercampusmvp/fixedguid   "dotnet FixedGuid.dll"   5 seconds ago   Up 2 seconds   0.0.0.0:61795->80/tcp   escalado-fixedguid-1

    - Acceso desde otros contenedores
        # docker-compose.yaml
        services:
            fixedguid:
                image: dockercampusmvp/fixedguid
                ports:
                    - 80
            proxy:
                image: dockercampusmvp/nodejs-sample
                ports:
                    - 5000:3000
        
        $ curl http://localhost:5000/get?url=http://fixedguid/api/greetings
            Nos devuelve siempre el mismo ID que es el contenedor que está arriba
        
        - Ahora escalamos fiedguid
            $ docker compose up -d --scale fixedguid=5
                PS C:\jlparres\GIT\Docker\02-Multicontenedores\escalado> docker ps
                CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS          PORTS                     NAMES
                f1a9139d959b   dockercampusmvp/nodejs-sample   "/bin/sh -c 'node se…"   15 seconds ago   Up 15 seconds   0.0.0.0:5000->3000/tcp    escalado-proxy-1
                9c7276713d66   dockercampusmvp/fixedguid       "dotnet FixedGuid.dll"   4 minutes ago    Up 4 minutes    127.0.0.1:32768->80/tcp   escalado-fixedguid-4
                dad9f7a338ef   dockercampusmvp/fixedguid       "dotnet FixedGuid.dll"   4 minutes ago    Up 4 minutes    127.0.0.1:32769->80/tcp   escalado-fixedguid-5
                dedd88fd73d3   dockercampusmvp/fixedguid       "dotnet FixedGuid.dll"   4 minutes ago    Up 4 minutes    127.0.0.1:32770->80/tcp   escalado-fixedguid-2
                103f75ccad07   dockercampusmvp/fixedguid       "dotnet FixedGuid.dll"   4 minutes ago    Up 4 minutes    127.0.0.1:32771->80/tcp   escalado-fixedguid-3
                313c666b6210   dockercampusmvp/fixedguid       "dotnet FixedGuid.dll"   4 minutes ago    Up 4 minutes    127.0.0.1:32772->80/tcp   escalado-fixedguid-1
        $ curl http://localhost:5000/get?url=http://fixedguid/api/greetings
            Ahora nos devuelve distintos ids, hasta un máximo de 5, porque tenemos 5 contenedores levantados.







