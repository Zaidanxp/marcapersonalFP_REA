# 4.3. Schema Builder

Una vez creada una migración tenemos que completar sus métodos `up()` y `down()` para indicar la tabla que queremos crear o el campo que queremos modificar. En el método `down()` siempre tendremos que añadir la operación inversa, eliminar la tabla que se ha creado en el método `up()` o eliminar la columna que se ha añadido. Esto nos permitirá deshacer migraciones dejando la base de datos en el mismo estado en el que se encontraban antes de que se añadieran.

Para especificar la tabla a crear o modificar, así como las columnas y tipos de datos de las mismas, se utiliza la clase `Schema`. Esta clase tiene una serie de métodos que nos permitirá especificar la estructura de las tablas independientemente del sistema de base de datos que utilicemos.
Crear y borrar una tabla

Para añadir una nueva tabla a la base de datos se utiliza el siguiente constructor:

```
Schema::create('users', function (Blueprint $table) {
    $table->increments('id');
});
```

Donde el primer argumento es el nombre de la tabla y el segundo es una función que recibe como parámetro un objeto del tipo `Blueprint` que utilizaremos para configurar las columnas de la tabla.

En la sección down de la migración tendremos que eliminar la tabla que hemos creado, para esto usaremos alguno de los siguientes métodos:

```
Schema::drop('users');
```
```
Schema::dropIfExists('users');
```

Al crear una migración con el comando de _Artisan_ `make:migration` ya nos viene este código añadido por defecto, la creación y eliminación de la tabla que se ha indicado y además se añaden un par de columnas por defecto (`id` y `timestamps`).

## Añadir columnas

El constructor `Schema::create` recibe como segundo parámetro una función que nos permite especificar las columnas que va a tener dicha tabla. En esta función podemos ir añadiendo todos los campos que queramos, indicando para cada uno de ellos su tipo y nombre, y además, si queremos, también podremos indicar una serie de modificadores como _valor por defecto_, _índices_, etc. Por ejemplo:

```
Schema::create('users', function($table)
{
    $table->increments('id');
    $table->string('username', 32);
    $table->string('password');
    $table->smallInteger('votos');
    $table->string('direccion');
    $table->boolean('confirmado')->default(false);
    $table->timestamps();
});
```

`Schema` define muchos tipos de datos que podemos utilizar para definir las columnas de una tabla, algunos de los principales son:

Comando	| Tipo de campo
--------|--------------
$table->boolean('confirmed'); | BOOLEAN
$table->enum('choices', array('foo', 'bar')); | ENUM
$table->float('amount'); | FLOAT
$table->increments('id'); | Clave principal tipo INTEGER con Auto-Increment
$table->integer('votes'); | INTEGER
$table->mediumInteger('numbers'); | MEDIUMINT
$table->smallInteger('votes'); |SMALLINT
$table->tinyInteger('numbers'); | TINYINT
$table->string('email'); | VARCHAR
$table->string('name', 100); | VARCHAR con la longitud indicada
$table->text('description'); | TEXT
$table->timestamp('added_on'); | TIMESTAMP
$table->timestamps(); | Añade los timestamps "created_at" y "updated_at"
->nullable() | Indicar que la columna permite valores NULL
->default($value) | Declare a default value for a column
->unsigned() | Añade UNSIGNED a las columnas tipo INTEGER

Los tres últimos se pueden combinar con el resto de tipos para crear, por ejemplo, una columna que permita nulos, con un valor por defecto y de tipo unsigned.

Para consultar todos los tipos de datos que podemos utilizar podéis consultar la documentación de Laravel en:

[http://laravel.com/docs/migrations#columns](http://laravel.com/docs/migrations#columns)

## Añadir índices

`Schema` soporta los siguientes tipos de índices:

Comando | Descripción
--------|------------
$table->primary('id'); | Añadir una clave primaria
$table->primary(array('first', 'last')); | Definir una clave primaria compuesta
$table->unique('email'); | Definir el campo como UNIQUE
$table->index('state'); | Añadir un índice a una columna

En la tabla se especifica como añadir estos índices después de crear el campo, pero también permite indicar estos índices a la vez que se crea el campo:

```
$table->string('email')->unique();
```

## Claves ajenas

Con `Schema` también podemos definir _claves ajenas_ entre tablas:

```
$table->bigInteger('user_id')->unsigned();
$table->foreign('user_id')->references('id')->on('users');
```

En este ejemplo, en primer lugar añadimos la columna `user_id` de tipo `UNSIGNED INTEGER` (siempre tendremos que crear primero la columna sobre la que se va a aplicar la clave ajena). A continuación, creamos la _clave ajena_ entre la columna `user_id` y la columna `id` de la tabla `users`.

    La columna con la clave ajena tiene que ser del mismo tipo que la columna a la que apunta. Si, por ejemplo, creamos una columna a un índice _auto-incremental_ tendremos que especificar que la columna sea `unsigned` para que no se produzcan errores.

También podemos especificar las acciones que se tienen que realizar para `on delete` y `on update`:

```
$table->foreign('user_id')
      ->references('id')->on('users')
      ->onDelete('cascade');
```

Para eliminar una _clave ajena_, en el método `down()` de la migración tenemos que utilizar el siguiente código:

```
$table->dropForeign('posts_user_id_foreign');
```

Para indicar la _clave ajena_ a eliminar tenemos que seguir el siguiente patrón para especificar el nombre `<tabla>_<columna>_foreign`. Donde "tabla" es el nombre de la tabla actual y "columna" el nombre de la columna sobre la que se creo la _clave ajena_.
