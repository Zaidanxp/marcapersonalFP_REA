# 2.3. Rutas

Las rutas de nuestra aplicación aplicación se tienen que definir en el fichero `routes/web.php`. Este es el punto centralizado para la definición de rutas y cualquier ruta no definida en este fichero no será válida, generado una excepción (lo que devolverá un error **404**).

Las rutas, en su forma más sencilla, pueden devolver directamente un valor desde el propio fichero de rutas, pero también podrán generar la llamada a una vista o a un controlador. Empezaremos viendo el primer tipo de rutas y en secciones posteriores se tratará como enlazarlas con una vista o con un controlador.

## Métodos de petición HTTP

**HTTP** define un conjunto de métodos de petición para indicar la acción que se desea realizar para un **recurso** determinado. Aunque estos también pueden ser sustantivos, estos métodos de solicitud a veces son llamados verbos **HTTP**.

- `GET`: 
    El método GET solicita una representación de un recurso específico. Las peticiones que usan el método GET sólo deben recuperar datos.
- `POST`: 
    El método POST se utiliza para enviar una entidad a un recurso en específico, causando a menudo un cambio en el estado o efectos secundarios en el servidor.
- `PUT`: 
    El modo PUT reemplaza todas las representaciones actuales del recurso de destino con la carga útil de la petición.
- `DELETE`: 
    El método DELETE borra un recurso en específico.

Existen otros métodos que, de momento, no abordaremos en esta documentación: `HEAD`, `CONNECT`, `OPTIONS`, `TRACE` y `PATCH`.

* _Información extraída de [mozilla.org](https://developer.mozilla.org/es/docs/Web/HTTP/Methods)_

## Rutas básicas

Las rutas, además de definir la _URL_ de la petición, también indican el **método** con el cual se ha de hacer dicha petición. Los dos métodos más utilizados y que empezaremos viendo son las peticiones tipo `GET` y tipo `POST`. Por ejemplo, para definir una petición tipo GET tendríamos que añadir el siguiente código a nuestro fichero `routes/web.php`:

```
Route::get('hola', function()
{
    return '¡Hola mundo!';
});
```

Este código se lanzaría cuando se realice una petición tipo `GET` a la ruta raíz de nuestra aplicación. Si estamos trabajando en local esta ruta sería `http://localhost:8000/hola` pero cuando la web esté en producción se referiría al dominio principal, por ejemplo: `http://www.dirección-de-tu-web.com/hola`. Es importante indicar que si se realiza una petición tipo `POST` o de otro tipo que no sea `GET` a dicha dirección se devolvería un error ya que esa ruta no está definida.

Para definir una ruta tipo `POST` se realizaría de la misma forma pero cambiando el verbo `GET` por `POST`:

```
Route::post('foo/bar', function()
{
    return '¡Hola mundo!';
});
```

En este caso la ruta apuntaría a la dirección URL `foo/bar` (`http://localhost:8000/foo/bar` o `http://www.dirección-de-tu-web.com/foo/bar`).

De la misma forma podemos definir rutas para peticiones tipo `PUT` o `DELETE`:

```
Route::put('foo/bar', function () {
    //
});
```

```
Route::delete('foo/bar', function () {
    //
});
```

Si queremos que una ruta se defina a la vez para varios verbos lo podemos hacer añadiendo un array con los tipos, de la siguiente forma:

```
Route::match(array('GET', 'POST'), '/', function()
{
    return '¡Hola mundo!';
});
```

O para cualquier tipo de petición HTTP utilizando el método any:

```
Route::any('foo', function()
{
    return '¡Hola mundo!';
});
```

## Añadir parámetros a las rutas

Si queremos añadir parámetros a una ruta simplemente los tenemos que indicar entre llaves `{}` a continuación de la ruta, de la forma:

```
Route::get('saluda/{nombre}', function($nombre)
{
    return '¡Hola ' . $nombre . '!';
});
```

En este caso estamos definiendo la ruta `/saluda/{nombre}`, donde _nombre_ es requerido y puede ser cualquier valor. En caso de no especificar ningún _nombre_ se produciría un error. El parámetro se le pasará a la función, el cual se podrá utilizar (como veremos más adelante) para por ejemplo obtener datos de la base de datos, almacenar valores, etc.

Para indicar que un parámetro es opcional añadiremos el símbolo `?` al final (y en este caso no daría error si no se realiza la petición con dicho parámetro):

```
Route::get('saluda/{nombre?}', function($nombre = null)
{
    return '¡Hola ' . $nombre . '!';
});
```

También podemos poner algún valor por defecto.

```
Route::get('saluda/{nombre?}', function($nombre = 'colega')
{
    return '¡Hola ' . $nombre . '!';
});
```

Laravel también permite el uso de expresiones regulares para validar los parámetros que se le pasan a una ruta. Por ejemplo, para validar que un parámetro esté formado solo por letras o solo por números:

```
Route::get('saluda/{nombre}', function($nombre)
{
    return '¡Hola ' . $nombre . '!';
})
->where('nombre', '[A-Za-z]+');
```

```
Route::get('user/{id}', function($id)
{
    //
})
->where('id', '[0-9]+');
```

Si hubiera varios parámetros podríamos validarlos usando un array:

```
Route::get('user/{id}/{name}', function($id, $name)
{
    //
})
->where(array('id' => '[0-9]+', 'name' => '[A-Za-z]+'))
```

## Agrupar rutas

El agrupamiento de rutas permite compartir atributos entre las rutas del grupo.

Los atributos que se pueden compartir entre las rutas de un grupo pueden ser:

- [Middlewares](https://laravel.com/docs/routing#route-group-middleware)
- [Controladores](https://laravel.com/docs/10.x/routing#route-group-controllers)
- [Subdominios](https://laravel.com/docs/10.x/routing#route-group-subdomain-routing)
- [Prefijos](https://laravel.com/docs/10.x/routing#route-group-prefixes)
- [Prefijos de nombre](https://laravel.com/docs/10.x/routing#route-group-name-prefixes)

A estas alturas es difícil comprender algunos de esos atributos, aunque sí podemos mostrar un ejemplo de prefijos de rutas.

### Prefijos de Rutas

La posibilidad de crear prefijos de rutas permite asociar un valor constante antes de cada una de las rutas de un grupo. Los dos siguientes códigos generan las mismas rutas:

```
Route::get('/proyectos', function () {
    //
});
Route::get('/proyectos/show/{id}', function () {
    //
});
```

```
Route::prefix('proyectos')->group(function () {
    Route::get('/', function () {
        //
    });
    Route::get('/show/{id}', function ($id) {
        //
    });
});
```

## Generar una ruta

Cuando queramos generar la _URL_ hasta una ruta podemos utilizar el siguiente método:

`$url = url('foo');`

Con este método nos aseguraremos que la _URL_ sea válida y además se le añadirá el dominio que tengamos definido en los ficheros de configuración. En general no será necesaria su utilización y simplemente podremos escribir la ruta a mano hasta una dirección de la forma: `/foo` (anteponiendo la barra `/` para asegurarnos que la ruta sea a partir de la raíz del dominio de nuestro sitio). Sin embargo se recomienda la utilización de este método en general para evitar problemas de generación de rutas no existentes o relativas (si se nos olvidase anteponer la `/`).

## Ejercicios:

A estas alturas ya tendríamos que ser capaces de añadir contenido estático a nuestra web, simplemente modificando el fichero de rutas y devolviendo todo el contenido desde ese fichero. Para evitar tener que mantener un inmenso fichero `routes/web.php` con todo el código mezclado en el mismo archivo, en las siguientes secciones separaremos el código de las vistas y más adelante añadiremos los controladores.

En este ejercicio vamos a definir las rutas principales que va a tener nuestro sitio web.

Necesitaremos crear las vistas para poder realizar las operaciones **CRUD** sobre cada una de las tablas. De momento, las vistas únicamente deben devolver el texto con la _operación/visualización_ que deberán realizar en un futuro.

Las siguientes son las pantallas principales y un ejemplo del resultado del **CRUD** sobre la tabla 

Método | Ruta | Texto a mostrar
-------|------|-------
GET | `/` | Pantalla principal
GET | `login` | Login usuario
GET | `logout` | Logout usuario
GET | `catalog` | Listado proyectos
GET | `catalog/show/{id}` | Vista detalle proyecto {id}
GET | `catalog/create` | Añadir proyecto
GET | `catalog/edit/{id}` | Modificar proyecto {id}
GET | `perfil/{id}` | Visualizar el currículo de {id}

Debemos asegurarnos de que todos los parámetros `{id}` sean números naturales.

El parámetro `{id}` es opcional. En el caso de que exista debe mostrar _Visualizar el currículo de_ y el número enviado, mientras que en caso de no enviar ningún valor para ese parámetro se debería mostrar _Visualizar el currículo propio_
