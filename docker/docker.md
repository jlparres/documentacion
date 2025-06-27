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
    $ docker volume rm <my-volumen>

    Eliminar todos los volumenes 
    $ docker volume prune

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

### Ejemplo práctico
    - Crear un servidor Wordpress con su base de datos MySQL
    - Después mantener persistente los volumenes

    # docker-compose.yaml
        services:
        mysql:
            image: mysql:8.0.3
            environment:
            - MYSQL_ROOT_PASSWORD=rootpassword
            - MYSQL_DATABASE=db_example
            - MYSQL_USER=adminuser
            - MYSQL_PASSWORD=adminpass
        volumes:
            - ./tmp:/var/lib/mysql

        wordpress:
            image: wordpress:4.9.1-apache
            environment:
                - WORDPRESS_DB_HOST=mysql   # Referenciar con el nombre del servicio
                - WORDPRESS_DB_USER=adminuser
                - WORDPRESS_DB_PASSWORD=adminpass
                - WORDPRESS_DB_NAME=db_example
            ports:
                - 9500:80

## Conceptos avanzados de los contenedores

### Redes de Docker
    - Ver las redes creadas por los contenedores
        $ docker network ls
    - Las redes de los contenedores, no soportan DNS, pero las redes creadas por docker compose y propias, si montan un servidor dns propi, por eso no podemos llamar al conenedor por el nombre.
    - Creando una propia red tipo Bridge
        $ docker network create --driver bridge mi-red-casa

    - Redes tipo HOST
        Solo soportada por máquinas Linux
        $ docker run -d --network host dockercampusmvp/go-hello-world
        ahora podemos hacer un curl http://localhost:80
        porque el contenedor está usando el puerto 80 del host

    - Manejo de redes en Docker
        $ docker network ls 
        $ docker network rm <id-network>
        $ docker network prune

### Contextos en Docker
    - Podemos tener acceso a varios contextos de instalaciones de Docker desde una misma máquina.
    - Tengo dos MVs con Docker corriendo, puedo definir un contexto para cada MV.

    - Listar contextos
        $ docker context ls
    
    - Crear contexto nuevo
        $ docker context create shared-vm --docker host=tcp:///shared-mv:2735
    


## Compilar Proyectos - Developer APPs
    - Ejecutar un proyecto de NetCore 2 sin necesidad de tenerlo instalado en la máquina
        Hay que copiar el proyecto en la raíz
        # Dockerfile
            FROM mcr.microsoft.com/dotnet/sdk:2.1
            WORKDIR /src
            COPY . .
            ENTRYPOINT [ "dotnet" ,"run" ] # Ejecutar el proyecto

        $ docker build . -t dotnet-run
        $ docker run dotnet-run

    - Publicar con dotnet run
        # Dockerfile
            FROM mcr.microsoft.com/dotnet/sdk:2.1
            WORKDIR /src
            COPY . .
            # Publicar el proyecto
            ENTRYPOINT [ "dotnet", "publish", "-o", "out" ]

        Con este método tenemos el problema que no tenemos los binarios, ya que se crean en el contenedor. Necesitamos un bind mount para recuperar la info.

    - Publicar con dotnet run y bind mount con el mismo Docker file
        $ docker run -v c/jlparres/Docker/dotnet:/src/out dotnet-run
        $ docker run -v $(pwd)/out:/src/out dotnet-run
    
    - Con el Dockerfile anterior y las anteriores instrucciones, tenemos un problema, que siempre que se quiera compilar el código hay que volver a crear la imagen, para ello eliminamos la instrucción COPY . . 
      e indicamos con bind mounts de donde hay que coger el código y donde tenemos que generar la compilación.
    
        # Dockerfile
            FROM mcr.microsoft.com/dotnet/sdk:2.1
            WORKDIR /src
            ENTRYPOINT [ "dotnet", "publish", "-o", "out" ]
        
        $ docker run -v C:\jlparres\GIT\Docker\03-Compilar\helloworld-netcore2:/src -v C:\jlparres\GIT\Docker\03-Compilar\helloword-app:/src/out dotnet-run

        El código de desarrollo está en la carpeta "helloworld-netcore2" y el código compilado en la carpeta "helloworld-app"

        - Compliar directamente sin tener un dockerfile. Ahora utilizamos directamente la imagen mcr.microsoft.com/dotnet/sdk:2.1 y lanzamos las instrucciones con bash -c "cd /src && dotnet publish -o out" todo lo demás es lo de antes.
        $ docker run -v C:\jlparres\GIT\Docker\03-Compilar\helloworld-netcore2:/src -v C:\jlparres\GIT\Docker\03-Compilar\helloword-app:/src/out mcr.microsoft.com/dotnet/sdk:2.1 bash -c "cd /src && dotnet publish -o out"

    - Crear proyectos con dotnet new mvc
        $ docker run -v C:\jlparres\GIT\Docker\03-Compilar\nuevo-proyecto\netcore21:/app mcr.microsoft.com/dotnet/sdk:2.1 /bin/bash -c "cd app && dotnet new mvc"
        $ docker run -v C:\jlparres\GIT\Docker\03-Compilar\nuevo-proyecto\net7:/app mcr.microsoft.com/dotnet/sdk:7.0 /bin/bash -c "cd app && dotnet new mvc"

### Utilizando Docker Compose para crear la compilación

### Ejercicio con Multi-stage
- Paso 1: Crear aplicación .NET
    $ docker pull mcr.microsoft.com/dotnet/sdk:6.0

    Crear un directorio en la máqiuna y situarnos en el
    $ docker run -v /Users/jlparres/Desktop/knowmadmood/GIT/Docker/multi-stage/ejercicioNET/ejercicio:/ejercicio mcr.microsoft.com/dotnet/sdk:6.0 dotnet new mvc -n ejercicioNET

### Depurar contenedores
    - Ver los logs 
        $ docker logs <id-contenedor>
    
    - Copiar ficheros de logs - Contenedor al host
        $ docker cp <id-contenedor>:logs/log.txt .
    - Copiar ficher de Host a Contenedor
        $ docker cp foo.txt <id-contenedor>:/app/foo.txt
    - Docker Commit -> Crear una imagen a partir de un contenedor en marcha - Necesita parar el contenedor para crear la imagen
        $ docker commit <id-contentedor-origen> <nombre-nueva-imagen>
    - Ahora podemos utilizar el nuevo contenedor para depurar y ver los ficheros
        $ docker run <nombre-nueva-imagen> /bin/bash


## Kubernetes

### Repaso de comandos
    $ docker images
    $ docker inspect <imagen>

    - Ver las capas que tiene la imagen y vemos los comandos de cada capa
    $ docker history <image>

    - docker tag
    $ docker tag eshop/webspa eshop/webspa:other-tag

    - Borrar imagenes
    $ docker rmi <imagen-id> -f

    - Docker Pull - Obtener imagenes
    $ docker pull repositorio/organizacion/node

    - Subir imagen a un repositorio, primero añadir un tag a la imagen y luego hacer el push
    $ docker tag node servidor.com/node:j1.1
    $ docker push servidor.com/node:j1.1

    - Login en el repositorio privado
    $ docker login servidor.com


    - Ver contenedores
    $ docker ps
    $ docker ps -a
    $ docker logs <contenedor-id>
    $ docker images
    $ docker run <imagen>
    $ docker top <conenedor-id>
    - Borrar todos los contenedores con los ids
        $ docker ps -q // muestra los ids 
    $ docker rm $(docker ps -a -q)

    - Redirección puertos
        El puerto 1433 interno del contenedor está redirigido al 5433 externo del contendor, nosotros accedemos por el 5433
    $ docker run -p 5433:1433 node
    $ docker run -p 5433:1433 -e "ACCEPT_EULA=Y" microsoft/mssql-server-linux -d 
    $ docker stop <contenedor-id>
    $ docker kill <contenedor-id>

    - Sesión interactiva
    $ docker exec -it <contenedor-id> /bin/sh

    - Copiar ficheros
    $ docker cp <contenedor-id>:/fichero.txt .
    $ docker cp fichero.txt <contenedor-id>:/mipath/fichero.txt


### Fundamentos de Kubernetes
    - Instalación de minikube
        $ curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-darwin-arm64
        $ sudo install minikube-darwin-arm64 /usr/local/bin/minikube
    - Ejecturar minikube
        $ minikube start
    - Interactuar con el cluster
        $ kubectl get po -A
        $ minikube kubectl -- get po -A
            $ alias kubectl="minikube kubectl --"
        $ minikube dashboard

    - Contexto de kubectl (cuando se tiene varios clusters instalados)
        $ kubectl config get-contexts
        $ kubectl get nodes --context minikube
    - Cambio de conteto 
        $ kubectl config use-context <nombre-contexto>
    
    - Despliegue de PODS con kubectl
        $ kubectl run my-first-deployment --image=dockercampusmvp/go-hello-world
    - Eliminar un POD
        $ kubectl delete pod my-first-deployment 

    - Crear deployments
        $ kubectl create deployment my-first-deployment --image=dockercampusmvp/go-hello-world

    - Obtener los deployments
        $ kubectl get deployments
        $ kubectl get deploy
    
    - Eliminar un deployment 
        $ kubectl delete deploy my-first-deployment

    - Resumen
        Un pod ejecuta uno o más contenedores
        No solemos crear pods directamente
        En su lugar creamos deployments, que indican a k8s qué (y cuántos) contenedores ejecutar. 
            Kubernetes creará los pods necesarios para asegurar el cumplimiento del deployment
        El comando `kubectl get pods` nos permite ver los pods
        El comando `kubectl get deployments` nos permite ver los deployments


    - Ejecicio
    $ kubectl run hello --image dockercampusmvp/go-hello-world
    $ kubectl run bb --image busybox --rm --restart=Never -it -- /bin/sh

    --restart={ Never, Always, oNFailure } - posibles valores

    - Exponer Servicios
        $ kubectl expose pod <nombre-pod> --name <nombre-servicio> --port <puerto>
        $ kubectl expose pod hello --name hello-svc --port 80

        - Obtener los servicios
        $ kubectl get svc

        - Asociar un servicio a un POD:
            $ kubectl run hello --image dockercampusmvp/go-hello-world
            $ kubectl expose pod hello --name hello-svc --port 80
            El pod "hello" utilizará siempre el servicio expuesto
    
    - Labels - Utilizadas para filtrar campos, que no tiene importancia para el POD
        $ kubectl label pod <nombre-pod> <var=value> --overwrite
        $ kubectl get pods --show-labels
    
#### Servicios y DNS
    $ kubectl run hello --image dockercampusmvp/go-hello-world
    $ kubectl run bb --image busybox --rm --restart=Never -it -- /bin/sh
        $ wget -qO- http://hello-svc
    $ kubectl delete pod hello
        $ wget -qO- http://hello-svc -> error porque no está el pod hello
    $ kubectl run hello2 --image dockercampusmvp/go-hello-world
        $ wget -qO- http://hello-svc -> error porque el pod hello2 no está asociado al servicio, debemos asociarlo con labels
    $ kubectl get pods --show-labels
    $ kubectl label pod hello2 run=hello --overwrite

#### Ciclo de vida de un POD
    > Succeeded: el pod se ha ejecutado correctamente
    > Running: el pod se está ejecutando
    > Failed: el pod se ha ejecutado con algún error
    > Pending: el pod debe empezar a ejecutarse (sus contenedores no han sido creados todavía)
    > Unknown: el estado del pod no se puede obtener. Esto es debido generalmente a algún error de comunicación con el nodo que hospeda el pod


#### Despliegues declarativos con ficheros .yaml
    Obtener la definición yaml de un pod / servicio 
    $ kubectl get svc hello-svc -o yaml

    Crear un namespace
    $ kubectl create ns <nombre-namespace>

    Eliminar un namespace (Elimina todo el contenido dentro del namespace)
    $ kubectl delete ns <espacio-de-nombres>

    Obtener las API-Versions del cluster de Kubernetes para la definición de ficheros .yaml
    $ kubectl api-versions

    Pasar de imperativo a declarativo
    $ kubectl run hello --image dockercampusmvp/go-hello-world --dry-run=client -o yaml 
    > Salida del yaml
        piVersion: v1
        kind: Pod
        metadata:
        creationTimestamp: null
        labels:
            run: hello
        name: hello
        spec:
        containers:
        - image: dockercampusmvp/go-hello-world
            name: hello
            resources: {}
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        status: {}

    Comando explain, se utiliza para ver las especificaciones de cualquier recurso de Kubernetes
    $ kubectl explain pod
    $ kubectl explain pod.spec
    $ kubectl explain pod.spec.containers
    $ kubectl explain pod.spec.containers.ports

    $ kubectl explain svc
    $ kubectl explain svc.spec
    $ kubectl explain svc.spec.type

    - Ejecicio práctico. Exponer un servicio para un pod
        Forma declarativa:
            $ kubectl expose pod hello --name hellosvc --port 80
            $ kubectl delete svc hellosvc
        Forma imperativa
            $ kubectl expose pod hello --name hellosvc --port 80 --dry-run -o yaml > svc-hello.yaml
            $ kubectl apply -f svc-hello.yaml

#### Dashboard del Clúster de Kuberentes
    En minikube tenemos el dashboard habilitado por defecto, solo necesitamos ejecutar lo siguiente
    $ minikube addons enable dashboard
    $ minikube addons enable metrics-server

    $ kubectl get svc,deploy,pod -n kubernetes-dashboard
```
    jlparres@MacBook-Pro-de-Jorge get-yaml % kubectl get svc,deploy,pod -n kubernetes-dashboard
    NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    service/dashboard-metrics-scraper   ClusterIP   10.106.97.118   <none>        8000/TCP   15d
    service/kubernetes-dashboard        ClusterIP   10.103.195.34   <none>        80/TCP     15d

    NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/dashboard-metrics-scraper   1/1     1            1           15d
    deployment.apps/kubernetes-dashboard        1/1     1            1           15d

    NAME                                             READY   STATUS    RESTARTS     AGE
    pod/dashboard-metrics-scraper-5d59dccf9b-x66ks   1/1     Running   2 (2d ago)   15d
    pod/kubernetes-dashboard-7779f9b69b-4k9t8        1/1     Running   4 (2d ago)   15d
```
    Ver el dashboard
    $ minikube dashboard


#### Port-forward
    Herramienta para acceder a cualquier pod haciendo un tunel TCP a un puerto del POD
    $ kubectl port-forward <nombre-pod> <puerto-local>:<puerto-pod>

#### Ejercicio Pod no responde, como redirigir las peticiones
    $ kubectl run -it --rm --restart=Never --image busybox bb -n ejercicio -- /bin/sh

    $ kubectl run hello2 --image dockercampusmvp/go-hello-world 

### Despliegue de aplicaciones
    - Replica Set
        Definir el número de replicas del pod que queremos
    
    - Ver replica Sets
        $ kubectl get rs
    
    - Escalar un Replica Set
        $ kubectl scale rs/hello --replicas=10
        $ kubectl scale rs/hello --replicas=2

    - Deployments
        Deployment es un objeto de "alto nivel" pensado, específicamente, para el despliegue de aplicaciones (recuerda, pods en nuestro contexto) sin estado. Al igual que ReplicaSet, un deployment permite ejecutar N pods (réplicas) todos ellos idénticos y garantizar que hay N en todo momento.

        jlparres@MacBook-Pro-de-Jorge ReplicaSet % kubectl apply -f deployment_rs.yaml 
        deployment.apps/hello created
        jlparres@MacBook-Pro-de-Jorge ReplicaSet % kubectl get pods
        NAME                    READY   STATUS    RESTARTS   AGE
        hello-7b44b6888-4ksl4   1/1     Running   0          7s
        hello-7b44b6888-75r66   1/1     Running   0          7s
        hello-7b44b6888-lvfnq   1/1     Running   0          7s
        hello-7b44b6888-pslcf   1/1     Running   0          7s
        hello-7b44b6888-q8nk2   1/1     Running   0          7s

        jlparres@MacBook-Pro-de-Jorge ReplicaSet % kubectl get rs
        NAME              DESIRED   CURRENT   READY   AGE
        hello-7b44b6888   5         5         5       75s

        Obtener los deployments
        $ kubectl get deploy
            NAME    READY   UP-TO-DATE   AVAILABLE   AGE
            hello   5/5     5            5           2m35s

    - Ver la salida en tiempo de ejecución
        $ kubectl get pods -o jsonpath="Name: {.metadata.name} Status: { .status.phase } ContainerStatuses[0].state..reason}" --watch


### Rolling Updates
    Crear deployment
        $ kubectl apply -f deployment.yaml
    
    Ver los deployments
        $ kubectl get deploy

    Ver los Replica Set
        $ kubectl get rs

    Ver los pods
        $ kubectl get pods

    Ver el historial de los deployments
        $ kubectl rollout history deploy/hello

    Ver el status del rolling update
    $ kubectl rollout status deploy/hello
        Waiting for deployment "hello" rollout to finish: 1 out of 2 new replicas have been updated...

    Rollback del Rolling Update
        - Vemos las versiones que hay
            $ kubectl rollout history deploy/hello
                deployment.apps/hello 
                REVISION  CHANGE-CAUSE
                1         Primera versión
                2         Actualizar versión 'invalid'
                3         Actualizar versión 'invalid2'
        - Seleccionamos la versión a la que queremos ir
            $ kubectl rollout undo deploy/hello --to-revision 1
        - Revisamos los replica set y vemos que todas las versiones se han desescalado a replica = 0 menos la del rollback
            $ kubectl get rs
                NAME                    DESIRED   CURRENT   READY   AGE
                explode-v1-77dd9fd85d   1         1         1       4d
                hello-556686c6d7        0         0         0       4m57s
                hello-7b44b6888         2         2         2       11m
                hello-7c6889d7cc        0         0         0       2m53s
        - Revisamos el history y vemos que la versión 1 acaba de promocionar a la última versión. Esto es porque no se cean más replicaSet, sino que se promociona la versión del rollback a la última versión
            $ kubectl rollout history deploy/hello
                deployment.apps/hello 
                REVISION  CHANGE-CAUSE
                2         Actualizar versión 'invalid'
                3         Actualizar versión 'invalid2'
                4         Primera versión

### Configuración de Contentedores
    - Configs maps
        $ kubectl get cm

        apiVersion: v1
        kind: ConfigMap
        metadata:
            name: democonfig
        data:
            Api: http://campusmvp.es
            Log: Debug

    jlparres@MacBook-Pro-de-Jorge config-maps % kubectl get pods
    NAME                          READY   STATUS                       RESTARTS      AGE
    cfg                           0/1     CreateContainerConfigError   0             6s

    - Error porque no tiene configMap
        $ kubectl get cm

    - Ejecutar el pod 
        $ kubectl apply -f pod.yaml
        $ kubectl exec -it cfg /bin/sh

    - Utilizar envFrom en un pod es utilizar toda la referencia del configMap referenciado


    - Creando config Map a partir de un fichero en modo imperativo
        $ kubectl create cm config2 --from-file config.json=config.json
        
    - Creando config Map modo imperativo indicando a kubectl que te genere el yaml
        $ kubectl create cm config2 --from-file config.json=config.json -o yaml --dry-run=client
        apiVersion: v1
        data:
        config.json: |-
            {
                "api": {
                    "server": "http://campusmvp.es",
                    "version": "v.2.0"
                }
            }
        kind: ConfigMap
        metadata:
        creationTimestamp: null
        name: config2

### Secrets
    - Creaer un secreto
        $ kubectl create secret generic demosecret --from-literal API_KEY=9876 --from-literal LOG_LEVEL=Debug

    - Ver secrets
        $ kubectl get secret demosecret

    - Obtener el yaml del secret
        $ kubectl get secret demosecret -o yaml


### Logs
    $ kubectl logs <pod>

    Cuando tenemos varias replicas
    $ kubectl logs pod -l <etiqueta=valor>


### Indicar comando inicial al POD
    Podemos hacer que el pod ejecute un Script

    apiVersion: v1
    kind: Pod
    metadata:
        name: command
    spec:
        containers:
        - name: main
            image: alpine
            command: ["/bin/sh", "-c", "while true; do echo hello; sleep 10;done"]
    restartPolicy: Never

    - Con argumentos
apiVersion: v1
kind: Pod
metadata:
  name: command
spec:
  containers:
  - name: main
    image: alpine
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello; sleep 10;done"]
  restartPolicy: Never

### Exponer la App al exterior
    - Pod BusyBox
        $ kubectl run bb --image busybox -it --rm --restart Never -- /bin/sh
    
    - Ingress
        $ kubectl get ing
        $ kubectl describe ing <recurso-ingress>

    - Instalar ingres en minikube
        $ minikube addons enable ingress
        $ kubectl get svc -n ingress-nginx

## PODS a Fondo
### HEALTHCHECK
    Dokerfile
        FROM dockercampusmvp/demoprobes
        RUN apt-get update && apt-get install curl -y
        HEALTHCHECK CMD curl --fail http://localhost || exit 1


### Recursos
    Describe minikube
    $ kubectl describe node minikube
    Top
    $ kubectl top nodes
    Escalar pods 
    $ kubectl scale deploy/hello-world-res --replica=2

### POD en varios contenedores
    Obtener la info:
    $ kubectl get pod dos-contenedores -o jsonpath="{range .status.containerStatuses[*]}{'CONTAINER:'}{@.name} {'READY:' }{@.ready} {'RESTARTS'}:{@.restartCount}{'\n'}{end}"

    Copiar un fichero de un pod a otro
    $ kubectl cp <fichero-orign> <nombre-pod>:<fichero-destino>
    $ kubectl cp <namespace>/<nombre-pod>:<ruta-pod> <directorio-local>

    Comando debug
    $ kubectl debug -it --target=<contenedor> --image <imagen-contenedor-efímero> <pod>







