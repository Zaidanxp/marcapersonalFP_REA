# 2.1 Instalación

Existen diferentes maneras de generar un proyecto Laravel. En las siguientes secciones veremos 2 de esas formas:
- [Utilizando Composer](#utilizando-composer)
- [Utilizando Laradock](#utilizando-laradock)

A lo largo del curso utilizaremos mayoritariamente la que hace uso de Laradock.

En cualquier caso, para almacenar los proyectos de Laravel que vayamos generando, crearemos un directorio `laravel` en el directorio `Documentos`.

```
cd ~/Documentos
mkdir laravel
cd laravel/
```

Y nos aseguraremos de tener actualizado Composer:
`sudo composer self-update --2`

## Utilizando Composer

Antes de poder utilizar Laravel, debemos asegurarnos de cumplir sus requisitos:

- PHP
- [Composer](https://getcomposer.org/)
- [Node y NPM](https://nodejs.org/)

A continuación, instalaremos Laravel installer utilizando composer, el cual ya tenemos instalado en la máquina virtual.

`composer global require "laravel/installer"`

Nos debemos asegurar de que el ejecutable de laravel sea accesible desde terminal de comandos. Para ello, editaremos el archivo ~/.bashrc e incluiremos, al final del mismo, la siguiente línea:

`alias laravel='~/.config/composer/vendor/bin/laravel'`

Como ya hemos hecho en anteriores ocasiones, podemos cerrar el terminal y volver a abrirlo, para que tenga efecto la línea anterior o, simplemente, ejecutar el siguiente comando:

`source ~/.bashrc` 

En este momento, ya podremos utilizar laravel para crear nuestra primera aplicación web:

`laravel new prueba`

La aplicación web creada con Laravel se encontrará en el directorio `prueba`.

`cd prueba`

En el entorno de desarrollo, podemos usar el servidor web que nos proporciona el propio Laravel a través de su _CLI_ `artisan`.

`php artisan serve`

El resultado lo podremos comprobar si accedemos a la dirección https://localhost:8000, a través de un navegador.

## Utilizando Laradock

### Estructura de carpetas

Una vez completados todos los pasos, la estructura quedará así:

```.
└── laravel
    ├── app
    └── laradock
```


### Descargar Laradock

1. Clonar el repositorio:

    `git clone https://github.com/Laradock/laradock.git`

    Para que funcione, tiene que estar instalado el [cliente de línea de comandos de Git](https://git-scm.com/downloads).

2. Copiar el fichero `.env.example` a `.env`:

    `cd laradock && cp .env.example .env && cd ..`

3. Editar el fichero `.env` de la carpeta laradock:

    - Modificar la ruta de la aplicación para que apunte a la carpeta laradock poniendo `APP_CODE_PATH_HOST=../app`
    - Si disponemos de más de una instalación de Laradock, modificar la variable COMPOSE_PROJECT_NAME y asignarle un nombre único para que los contenedores tengan nombres diferentes.
    - Seleccionar la versión de PHP: `PHP_VERSION=8.2`
    - Modificar el driver de base de datos de phpMyAdmin: `PMA_DB_ENGINE=mariadb`

### Nuevo proyecto de Laravel

#### Generar el proyecto

**atención** _En estos comandos, si se ha renombrado `app`, cambiar solo la última ocurrencia, después de `laravel/laravel`._

```
docker run -it --rm --name php-cli \
    -v "$PWD:/usr/src/app" thecodingmachine/php:8.2-v4-slim-cli \
    composer create-project --prefer-dist laravel/laravel app
```

#### (Re)arrancar los contenedores

Los comandos de `docker-compose` se lanzan desde la carpeta `laradock`.

#### Arrancar los contenedores necesarios:

`docker compose up -d nginx mariadb phpmyadmin workspace`

Y para reiniciar un contenedor concreto:

`docker compose restart nginx`

Crear la base de datos

1. Acceder a [phpMyAdmin](http://localhost:8081/)

    - Servidor mariadb y usuario root/root.
    - Crear la base de datos app y el usuario app/app.

2. Editar el `.env` de la aplicación

```
    DB_CONNECTION=mysql
    DB_HOST=mariadb
    DB_PORT=3306
    DB_DATABASE=app
    DB_USERNAME=app
    DB_PASSWORD=app
```

#### Acceder al sitio web

Página principal: http://localhost

![Captura-de-pantalla-2020-11-15-a-las-11-21-17](https://bootcamp.laravel.com/img/screenshots/fresh-dark.png)
