### Bajar una imagen
>docker image pull [nombre_imagen] #usa latest por default
- [listado de imagenes](https://hub.docker.com/)

### Crear un contenedor
> docker container create --name [debian] [imagenContenedor] # Crear contenedor

<code>
si le doy la opcion -i (interactive al crear) es para mantener abierta la stdin(input) <br>
el -i(interactive al start) es para poder usar esa stdin
</code>

### Detalle de un comando
> docker [comando] --help

### Listar contenedores
>docker ps -a # lista todos los contenedores

### Iniciar un contenedor ya creado
> docker container start [container name]

### Copiar un archivo desde el host al contenedor
> docker cp [ruta archivo] [nombre_container]:[ruta destino archivo]

### Copiar un archivo desde un contenedor al host
> docker cp [nombre_container]:[ruta archivo] [ruta destino archivo]  

### Metadata del contenedor
> docker container inspect [container_name]

### Inicializar un contenedor desde una imagen no pulleada o ya existente (docker images)
> docker run -it --name [container_name] [imagenContenedor]<br>
<code>Cada vez que ejecuto docker run, crea un contenedor nuevo, no confundir con docker start</code>
<code> Puedo crear un contenedor con el parametro --rm y esto hara que se cree el contenedor y cuando este se detenga se elimine automaticamente</code>

### crear una imagen a partir de un Dockerfile
> docker build -t [empresa]/app:[version_app o tag] [ruta del directorio donde esta el dockerfile. example ./]


### Ejecutar un comando en un contenedor que esta en marcha
> docker container exec [container_params] [container_name] [comando]

### Listado de procesos de un container
> docker top [container_name]

### recuperar el -it o consola de un contenedor corriendo
> docker attach [nombre o id]

### Abrir otra instancia -it de un contenedor
> docker exec -it [container_name] [bash]


### Recuperar un tag (darle un tag a una imagen dangling)
> basicamente, si recupero el contenido integro del dockerfile en el momento que genere esa imagen y asigno un nuevo tag, este se asignara a la imagen correspondiente que esta dangling 

## Detener contenedores

    Parar un container desde fuera soft way
    > docker stop [nombre o id]

    Parar un container hard way desde fuera (not recommend)
    > docker kill [nombre o id]

    detener varios contenedores a la vez
    > docker stop [nombre o id] [nombre o id] [nombre o id] [nombre o id] [nombre o id]

    detener todos los contenedores
    > docker stop $(docker ps -q)<br>


## filtrar contenedores por nombre
    > docker ps -q lista las id's de todos los contenedores corriendo
    > docker ps -f "name=debian-*" -q<br>
    > docker ps -qf "name=debian-*"
    listara los ids de los contenedores que tengan "debian-"en su nombre

    filtrar contenedores que tengan en comun una imagen
    > docker ps -qf "ancestor=[nombre_imagen]"

    Eliminar contenedores que tengan en comun una imagen
    > docker stop $(docker ps -qf "ancestor=[nombre_imagen]")

    detener todos los contenedores
    > docker stop $(> docker ps --filter "name=debian-*" -q)

## Eliminar contenedores (todos deben estar parados)
    > docker remove [nombre o id]

    Eliminar varios
    > docker remove [nombre o id] [nombre o id] [nombre o id] [nombre o id]

    Eliminar todos los contenedores
    > docker rm $(docker ps -aq)

    Eliminar contenedores filtrados por nombre
    > docker rm $(docker ps -aqf "name=debian-*")

    Eliminar contenedores filtrados por imagen
    > docker rm $(docker ps -aqf "ancestor=debian")

    Eliminar contenedores parados
    > docker container prune

    Forzar eliminacion de un contenedor
    > docker rm -f [nombre o id]
    -f(force)
    
## Eliminar imagenes (no debe haber contenedores usando esas img o hay que forzar)
    Eliminar una imagen
    > docker image rmi [nombre o id]

    Borrar todas las imagenes
    > docker rmi $(docker images -q)

    Eliminar imagenes que tienen contenedores en marcha (borra nombre de imagen y deja su id (dangling))
    > docker rmi -f [nombre]

    Listar imagenes eliminadas con contenedor corriendo (sin nombre)
    > docker images -f "dangling=true"

    Borra imagenes dangling que estan en desuso
    > docker image prune

    Borra imagenes que no tengan contenedores, ya sean dangling o no
    > docker image prune --all

## Elimina todos los contenedores parados, redes no usadas, todas las imagenes dangling
> docker system prune

## Identificar un contenedor desde otro contenedor
    Inspeccionar contenedor
    > docker inspect [nombre_contenedor]
    buscar la seccion de Networking y IPAddress

    - Por defecto docker crea una red llamada Bridge donde todos los contenedores que se van creando se les va asignando una ip una vez se conectan a esta red por defecto, por ende por defecto todos los contenedores se encontraran en la misma red (Bridge)

    Una forma de filtrar las redes dentro de todo el inspect
    > docker inspect --format "{{.NetworkSettings.IPAddress}}" [nombre_contenedor]
    > docker inspect -f "{{.NetworkSettings.IPAddress}}" [nombre_contenedor] #-f de format

    -Actualizo la ip del servidor a la ip del contenedor y como estan en la misma red ya podre ver el servicio corriendo en la ip del contenedor:[puerto] desde otro contenedor o el host inclusive

<hr>

### Docker overlay 2

- [reference](https://docs.docker.com/storage/storagedriver/overlayfs-driver/)
- Lower dir: carpetas de la imagen
- Merged dir: mezcla de carpetas (contenedor)
- Upper dir: diferencias del contenedor

![layers](https://docs.docker.com/storage/storagedriver/images/overlay_constructs.jpg)

## Docker Bridge
    Crear un contenedor con puertos publicados
    docker run -itw /app -p [puerto host]:[puerto container] --name [container_name] [img_name]:[img_ver]

![layers](https://i.postimg.cc/pdjvBdmV/docker-Bridge.png)

## Dockerizar una app (Dockerfile)

    Crear el Dockerfile (consultar seccion de dockerfile)
    > docker build -t [empresa]/app:[version_app o tag] [ruta del directorio donde esta el dockerfile]
    example:
    > docker build -t keaguirre/node:0.1.0 ./
    > docker run -it --name [container_name]

## Shell form vs Exec form
    # Exec form - ejecuta el comando directamente en node sin pasar por shell
    >CMD [ "node", "index.js"]

    # Shell form la señal pasa a traves de shell quien ejecuta el comando
    >CMD node index.js
    #CMD [command] of the console

    # shell form
        - hay 2 procesos (sh + tu proceso)
        - ctrl + c para que tu proceso cierre pq lo capta sh y envia sigint(input usr)
        - tu proceso no recibe sigterm cuando se usa docker stop
        - las variables de entorno se sustituyen por su valor

    # exec form
        - 1 proceso
        - ctrl + c no para el proceso (a menos que lo especifiques en tu codigo)
        - tu proceso si recibe sigterm cuando se usa docker stop
        - las variables de entorno no se sustituyen por su valor

## CMD(shell form) vs Entrypoint
    - CMD: es comando que se define por parte de la imagen como comando por defecto para los contenedores que se crean a partir de esta imagen.
    - ENTRYPOINT: ejecutable obligatorio que define la imagen. se define como ENTRYPOINT["comando"], este comando posteriormente no se puede reemplazar solo asignarle mas parametros.

    - Se pueden apilar entrypoint con cmd, para asignar comandos o parametros obligatorios con comandos o parametros por defecto o opcionales.

    -si se puede cambiar el comando de ejecucion de entrypoint al crear un contenedor siguiendo la sig estructura
    > docker run [params creacion contenedor] --entrypoint [comando o ejecutable] [imagen] [parametros para el comando entrypoint]

## [Bind mounts](https://docs.docker.com/storage/bind-mounts/)

Bind mounts have been around since the early days of Docker. Bind mounts have limited functionality compared to volumes. When you use a bind mount, a file or directory on the host machine is mounted into a container. The file or directory is referenced by its absolute path on the host machine. By contrast, when you use a volume, a new directory is created within Docker’s storage directory on the host machine, and Docker manages that directory’s contents.

The file or directory does not need to exist on the Docker host already. It is created on demand if it does not yet exist. Bind mounts are very performant, but they rely on the host machine’s filesystem having a specific directory structure available. If you are developing new Docker applications, consider using named volumes instead. You can’t use Docker CLI commands to directly manage bind mounts.

![bindmounts](https://docs.docker.com/storage/images/types-of-mounts-bind.png)


### Comandos curso <hr>
### creacion y ejecucion de contenedores
- docker image pull debian
- docker ps -a # lista todos los contenedores
- docker container create -i --tty --name debian-console debian
- docker start -i debian-console
- exit
- docker run -it --name fedora-container fedora
- docker start -i debian-console

-------------------------------------
- docker run -it -w /app --name 01-app-node ubuntu:22.04
