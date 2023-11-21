# 4.4. Modelos de datos mediante ORM

El _mapeado objeto-relacional_ (más conocido por su nombre en inglés, _Object-Relational Mapping_, o por sus siglas **ORM**) es una técnica de programación para convertir datos entre un lenguaje de programación orientado a objetos y una base de datos relacional como motor de persistencia. Esto posibilita el uso de las características propias de la orientación a objetos, podremos acceder directamente a los campos de un objeto para leer los datos de una base de datos o para insertarlos o modificarlos.

_Laravel_ incluye su propio sistema de **ORM** llamado _Eloquent_, el cual nos proporciona una manera elegante y fácil de interactuar con la base de datos. Para cada tabla de la base datos tendremos que definir su correspondiente modelo, el cual se utilizará para interactuar desde código con la tabla.

## Definición de un modelo

Por defecto los modelos se guardarán como clases _PHP_ dentro de la carpeta `app/Models`.

Para definir un modelo que use _Eloquent_ únicamente tenemos que crear una clase que herede de la clase `Model`:

```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Proyecto extends Model
{
    use HasFactory;
}
```

Sin embargo es mucho más fácil y rápido crear los modelos usando el comando `make:model` de _Artisan_:

```
php artisan make:model Proyecto
```

Este comando creará el fichero `Proyecto.php` dentro de la carpeta `app/Models` con el código básico de un modelo que hemos visto en el ejemplo anterior.

## Convenios en Eloquent

### Nombre

En general, el nombre de los modelos se pone en singular, con la primera letra en mayúscula, mientras que el nombre de las tablas suele estar en plural. Gracias a esto, al definir un modelo no es necesario indicar el nombre de la tabla asociada, sino que _Eloquent_ automáticamente buscará la tabla transformando el nombre del modelo a minúsculas y buscando su plural (en inglés). En el ejemplo anterior, que hemos creado el modelo `Proyecto` buscará la tabla de la base de datos llamada `proyectos` y en caso de no encontrarla daría un error.

Si la tabla tuviese otro nombre lo podemos indicar usando la propiedad protegida `$table` del modelo:

```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Proyecto extends Model
{
    use HasFactory;

    protected $table = 'mis_proyectos';
}
```

### Clave primaria

_Laravel_ también asume que cada tabla tiene declarada una _clave primaria_ con el nombre `id`. En el caso de que no sea así y queramos cambiarlo tendremos que sobrescribir el valor de la propiedad protegida `$primaryKey` del modelo, por ejemplo: `protected $primaryKey = 'my_id';`.

    Es importante definir correctamente este valor ya que se utiliza en determinados métodos de _Eloquent_, como por ejemplo para buscar registros o para crear las relaciones entre modelos.

### Timestamps

Otra propiedad que, en ocasiones, tendremos que establecer son los _timestamps_ automáticos. Por defecto, _Eloquent_ asume que todas las tablas contienen los campos `updated_at` y `created_at` (los cuales los podemos añadir muy fácilmente con `Schema` añadiendo `$table->timestamps()` en la migración). Estos campos se actualizarán automáticamente cuando se cree un nuevo registro o se modifique. En el caso de que no queramos utilizarlos (y que no estén añadidos a la tabla) tendremos que indicarlo en el modelo o de otra forma nos daría un error. Para indicar que no los actualice automáticamente tendremos que modificar el valor de la propiedad pública `$timestamps` a `false`, por ejemplo: `public $timestamps = false;`.

A continuación se muestra un ejemplo de un modelo de Eloquent en el que se añaden todas las especificaciones que hemos visto:

```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Proyecto extends Model
{
    use HasFactory;

    protected $table = 'mis_proyectos';
    protected $primaryKey = 'my_id'
    public $timestamps = false;
}
```

## Uso de un modelo de datos

Una vez creado el modelo, ya podemos empezar a utilizarlo para recuperar datos de la base de datos, para insertar nuevos datos o para actualizarlos. **El sitio correcto donde realizar estas acciones es en el controlador**, el cual se los tendrá que pasar a la vista ya preparados para su visualización.

Es importante que para su utilización indiquemos al inicio de la clase el _espacio de nombres_ del modelo o modelos a utilizar. Por ejemplo, si vamos a usar los modelos `User` y `Proyecto` tendríamos que añadir:

```
use App\Models\User;
use App\Models\Proyecto;
```

## Consultar datos

Para obtener todas las filas de la tabla asociada a un modelo usaremos el método `all()`:

```
$proyectos = Proyecto::all();

foreach( $proyectos as $proyecto ) {
    echo $proyecto->name;
}
```

Este método nos devolverá un _array_ de resultados, donde cada item del _array_ será una instancia del modelo `Proyecto`. Gracias a esto al obtener un elemento del _array_ podemos acceder a los campos o columnas de la tabla como si fueran propiedades del objeto ($proyecto->nombre).

    Nota: Todos los métodos que se describen en la sección de "Constructor de consultas" y en la documentación de _Laravel_ sobre "Query Builder" también se pueden utilizar en los modelos _Eloquent_. Por lo tanto podremos utilizar `where`, `orWhere`, `first`, `get`, `orderBy`, `groupBy`, `having`, `skip`, `take`, etc. para elaborar las consultas.

_Eloquent_ también incorpora el método find($id) para buscar un elemento a partir del identificador único del modelo, por ejemplo:

```
$proyecto = Proyecto::find(1);
echo $proyecto->nombre;
```

Si queremos que se lance una excepción cuando no se encuentre un modelo podemos utilizar los métodos `findOrFail()` o `firstOrFail()`. Esto nos permite capturar las excepciones y mostrar un error `404` cuando sucedan.

```
$model = Proyecto::findOrFail(1);
```

```
$model = Proyecto::where('votos', '>', 100)->firstOrFail();
```

A continuación se incluyen otros ejemplos de consultas usando _Eloquent_ con algunos de los métodos que ya habíamos visto en la sección "Constructor de consultas":

```
// Obtener 10 proyectos con más de 100 votos
$proyectos = Proyecto::where('votos', '>', 100)->take(10)->get();
```

```
// Obtener el primer proyecto con más de 100 votos
$proyecto = Proyecto::where('votos', '>', 100)->first();
```

También podemos utilizar los métodos agregados para calcular el **total** de registros obtenidos, o el **máximo**, **mínimo**, **media** o **suma** de una determinada columna. Por ejemplo:

$count = Proyecto::where('votos', '>', 100)->count();
$votos = Proyecto::max('votos');
$votos = Proyecto::min('votos');
$votos = Proyecto::avg('votos');
$total = Proyecto::sum('votos');

## Insertar datos

Para añadir una entrada en la tabla de la base de datos asociada con un modelo simplemente tenemos que crear una nueva instancia de dicho modelo, asignar los valores que queramos y por último guardarlos con el método `save()`:

```
$user = new User;
$user->name = 'Juan';
$user->save();
```

Para obtener el identificador asignado en la base de datos después de guardar (cuando se trate de tablas con índice _auto-incremental_), lo podremos recuperar simplemente accediendo al campo `id` del objeto que habíamos creado, por ejemplo:

```
$insertedId = $user->id;
```

## Actualizar datos

Para actualizar una instancia de un modelo es muy sencillo, solo tendremos que recuperar en primer lugar la instancia que queremos actualizar, a continuación modificarla y por último guardar los datos:

```
$user = User::find(1);
$user->email = 'juan@gmail.com';
$user->save();
```

## Borrar datos

Para borrar una instancia de un modelo en la base de datos simplemente tenemos que usar su método `delete()`:

```
$user = User::find(1);
$user->delete();
```

Si, por ejemplo, queremos borrar un conjunto de resultados también podemos usar el método `delete()` de la forma:

```
$affectedRows = Proyecto::where('votos', '>', 100)->delete();
```

## Más información

Para más información sobre como crear relaciones entre modelos, _eager loading_, etc. podéis consultar directamente la documentación de Laravel en:

[http://laravel.com/docs/eloquent](http://laravel.com/docs/eloquent)
