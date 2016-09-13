#Docker base para trabajar con drupal 8#

Este docker es una modificación del doker de [@keopx](https://github.com/keopx).

Esta pensada para trabajar tanto en OSX como en Gnu/linux

Necesitamos tener docker intalado en su versión estable.

Parar arrancar instalamos docker  [https://docker.com/](https://docs.docker.com/mac/)

Docker nos da tres herramientas:

* docker-machine: Es la encargada en mac y windows de virtualzar un host, lo suele hacer con Virtualbox, nosotros así lo aremos
* docker: es el comando principal con el que trabajar, apenas lo usaremos.
* docker-compose: Es la herramienta que nos permite orquestar distintos docers (contenedores) entre si para montar nuestra infraestructura

Nuestro docker trabaja con varios contenedores:

* Un contenedor con **Debian 8**, **Apache 2.4**y **PHP7**
* Un contenedor para la base de datos con: **Mysql 5.5**
* Un contenedor con **Phpmyadmin**
* Un contenedor con **MailHog**


## Instalación de nuestro entorno ##

Clonamos el repositorio y una vez dentro tenemos tenemos que hacer una serie de pasos

Levantar el virtualhost (Si estamos en mac)

`$ docker-machine start default`

Internamente necesitamos poder EXPORTAR la configuración,

esto lo hacemos con el comando

`docker-machine env`

Esto te dará la configuración completa, si ejecutas este comando te lo incluye

`eval $(docker-machine env)`


## Descarga y creación de nuestros contenedores ##

Docker-compose orquesta todos los pasos en orden para la generación de los contenedores y su realación lo hacemos con este comando y parametro. Le cuesta un rato, la primera vez.

`$ docker-compose build`

Una vez que a creado la imagen podemos levantar la maquina

`$ docker-compose up -d`

Si le quitamos el `-d` podremos ver el log y ver que escupen los contenedores.

## Run bash ##

Una vez levantada la maquina podemos ver los procesos con el comando:

`$ docker-compose ps`

Para poder accesder a la maquina necesitamos localizar el nombre del contender con el `_web_`

`docker exec -it dockerhal_web_1 /bin/bash`

sustituye  dockerhalt_web_1_ por  _name_  

`docker-compose ps`

## Configuración local##

Tenemos que incluir el unicef.local en el hosts, ejecutamos el comando

`$ docker-machine ip`

Modificamos el fichecro con sudo  _/etc/hosts_ new site _name_ con la ip que nos ha dado el fichero

`sudo echo "ip unicef.local" >> /etc/hosts`


## Apagar el entorno ##

`$ docker-compose down`

`$ docker-machine stop`
