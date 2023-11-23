# 4.2. Migraciones

Las migraciones son un sistema de _control de versiones para bases de datos_. Permiten que un equipo trabaje sobre una base de datos añadiendo y modificando campos, manteniendo un histórico de los cambios realizados y del estado actual de la base de datos. Las migraciones se utilizan de forma conjunta con la herramienta _Schema builder_ (que veremos en la siguiente sección) para gestionar el esquema de base de datos de la aplicación.

La forma de funcionar de las migraciones es crear ficheros (_PHP_) con la descripción de la tabla a crear y posteriormente, si se quiere modificar dicha tabla se añadiría una nueva migración (un nuevo fichero _PHP_) con los campos a modificar. _Artisan_ incluye comandos para crear migraciones, para ejecutar las migraciones o para hacer _rollback_ de las mismas (volver atrás).

## Crear una nueva migración

Para crear una nueva migración se utiliza el comando de _Artisan_ `make:migration`, al cual le pasaremos el nombre del fichero a crear y el nombre de la tabla:

```
php artisan make:migration create_estudiantes_table --create=estudiantes
```

Esto nos creará un fichero de migración en la carpeta `database/migrations` con el nombre `<TIMESTAMP>_create_estudiantes_table.php`. Al añadir un _timestamp_ a las migraciones el sistema sabe el orden en el que tiene que ejecutar (o deshacer) las mismas.

Si lo que queremos es añadir una migración que modifique los campos de una tabla existente tendremos que ejecutar el siguiente comando:

```
php artisan make:migration add_ciclo_to_estudiantes_table --table=estudiantes
```

En este caso se creará también un fichero en la misma carpeta, con el nombre `<TIMESTAMP>_add_ciclo_to_estudiante_table.php` pero preparado para modificar los campos de dicha tabla.

Por defecto, al indicar el nombre del fichero de migraciones se suele seguir siempre el mismo patrón (aunque el realidad el nombre es libre). Si es una migración que crea una tabla el nombre tendrá que ser `create_<table-name>_table` y si es una migración que modifica una tabla será `<action>_to_<table-name>_table`.

## Estructura de una migración

El fichero o clase _PHP_ generada para una migración siempre tiene una estructura similar a la siguiente:

```
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        //
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        //
    }
};
```

En el método `up()` es donde tendremos crear o modificar la tabla, y en el método `down()` tendremos que deshacer los cambios que se hagan en el `up()` (eliminar la tabla o eliminar el campo que se haya añadido). Esto nos permitirá poder ir añadiendo y eliminando cambios sobre la base de datos y tener un control o histórico de los mismos.

## Ejecutar migraciones

Después de crear una migración y de definir los campos de la tabla (en la siguiente sección veremos como especificar esto) tenemos que lanzar la migración con el siguiente comando:

```
php artisan migrate
```

    Si nos aparece el error _"class not found"_ lo podremos solucionar llamando a `composer dump-autoload` y volviendo a lanzar las migraciones.

Este comando aplicará la migración sobre la base de datos. Si hubiera más de una migración pendiente se ejecutarán todas. Para cada migración se llamará a su método `up()` para que cree o modifique la base de datos. Posteriormente, en caso de que queramos deshacer los últimos cambios podremos ejecutar:

```
php artisan migrate:rollback
```

O si queremos deshacer todas las migraciones

```
php artisan migrate:reset
```

Un comando interesante, cuando estamos desarrollando un nuevo sitio web, es `migrate:refresh`, el cual deshará todos los cambios y volver a aplicar las migraciones:

```
php artisan migrate:refresh
```

Además, si queremos comprobar el estado de las migraciones, para ver las que ya están instaladas y las que quedan pendientes, podemos ejecutar:

```
php artisan migrate:status
```
