# 5.3. Ejercicios

En los ejercicios de esta sección vamos a completar la gestión de proyectos, terminando el procesamiento de los formularios y añadiendo el sistema de autenticación de usuarios.

## Ejercicio 1 - Migración de la tabla usuarios (0.5 puntos)

En primer lugar vamos a comprobar la existencia de la tabla de la base de datos para almacenar los usuarios que tendrán acceso a la plataforma de gestión de proyectos.

Como hemos visto en la teoría, _Laravel_ ya incluye una migración con el nombre `create_users_table` para la tabla `users` con todos los campos necesarios. Vamos a abrir esta migración y a comprobar que los campos incluidos coinciden con los de la siguiente tabla, añadiendo los que no existan:

Campo | Tipo | Modificador
--|--|--
id | Autoincremental | 
name | String | 
email | String | unique
email_verified_at | timestamp | nullable
password | String | 
remember_token | Campo remember_token | 
timestamps | Timestamps de Eloquent |  

Vamos a crear una nueva migración para añadir los campos nombre y apellidos, que son comunes tanto para los estudiantes como para los docentes:

Campo | Tipo | Modificador
--|--|--
nombre | String | 50 
apellidos | String | 100

Para esto usamos el comando de _Artisan_ que crea las migraciones y editamos el fichero creado en `database/migrations`.

```bash
php artisan make:migration add_nombre_apellidos_to_users_table --table=users
```

> Renombra el fichero como _`[año_actual]`_`_12_12_000001_add_nombre_apellidos_to_users_table.php`.

```php
public function up()
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('nombre', 50)->after('name');
        $table->string('apellidos', 100)->after('nombre');
    });
}
```

Comprueba también que en el método `down()` de la migración se deshagan los cambios que se hacen en el método `up()`, en este caso sería eliminar los campos añadidos.

```php
public function down()
{
    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('nombre');
        $table->dropColumn('apellidos');
    });
}
```

Por último, usamos el comando de Artisan que añade las nuevas migraciones y comprobamos con PHPMyAdmin que la tabla contiene todos los campos indicados.

```bash
php artisan migrate
```

## Ejercicio 2 - Seeder de usuarios (0.5 puntos)

Ahora vamos a proceder a rellenar la tabla users con los datos iniciales. Para esto editamos el fichero de semillas situado en database/seeds/DatabaseSeeder.php y seguiremos los siguientes pasos:

    Creamos un método privado (dentro de la misma clase) llamado seedUsers() que se tendrá que llamar desde el método run de la forma:

public function run() {
    // ... Llamada al seed del catálogo

    self::seedUsers();
    $this->command->info('Tabla usuarios inicializada con datos!');
}

    Dentro del nuevo método seedUsers() realizamos las siguientes acciones:
        En primer lugar borramos el contenido de la tabla users.
        Y a continuación creamos un par de usuarios de prueba. Recuerda que para guardar el password es necesario encriptarlo manualmente usando el método bcrypt (Revisa la sección "Registro de un usuario").

Por último tendremos que ejecutar el comando de Artisan que procesa las semillas. Una vez realizado esto comprobamos en PHPMyAdmin que se han añadido los usuarios a la tabla users.
Ejercicio 3 - Sistema de autenticación (1 punto)

En este ejercicio vamos a completar el sistema de autenticación. En primer lugar ejecuta los comandos de Artisan para generar todas las rutas y vistas necesarias para el control de usuarios con Laravel/Breeze.

composer require laravel/ui --dev
php artisan ui vue --auth

A continuación edita el fichero routes/web.php y realiza las siguientes acciones:

    Elimina (o comenta) las rutas de login y logout que habíamos añadido manualmente en los primeros ejercicios a fin de que se utilicen las nuevas rutas definidas por Laravel.
    Añade un middleware de tipo grupo que aplique el filtro auth para proteger todas las rutas del catálogo (menos la raíz / y las de autenticación).
    Revisa mediante el comando de Artisan php artisan route:list las nuevas rutas y que el filtro auth se aplique correctamente.

Modifica la redirección que se realiza tras el login o el registro de un usuario para que redirija a la ruta /catalog. Para esto tienes que modificar la constante HOME en el fichero app/Providers/RouteServiceProvider.php, (revisa el apartado "Autenticación de un usuario" de la teoría).
Ejercicio 4 - Adaptar el layout (1 punto)

Con el cambio a la versión 8 de Laravel, vamos a utilizar dicho layout para las vistas del catálogo, lo que nos permitirá, posteriormente, utilizar las pantallas para VueJS.
navbar

Para continuar con la estructura que teníamos en el layout resources/views/layouts/master.blade.php, vamos a extraer del resources/views/layouts/app.blade.php el contenido que hay entre las etiquetas <nav> y </nav> para llevarlo al partial resources/views/partials/navbar.blade.php, sin borrar el contenido original, ya que nuestra intención es adaptarlo.

En el lugar que ocupaban las etiquetas <nav> y </nav>, referencia al partial navbar, tal y como estaba en el layout master: @include('partials.navbar').

Del navbar original, nos interesa trasladar, al nuevo navbar, el contenido existente entre las etiquetas <ul class="navbar-nav mr-auto"> y </ul>. Como, este contenido únicamente debe estar visible si el usuario está autenticado, las meteremos dentro de etiquetas de blade @guest, @elseguest y @endguest.

El resto del contenido del navbar original, lo podemos borrar.
layout app

Modifica las vistas del directorio catalog, además de la de logout, para que, en lugar de utilizar el layout resources/views/layouts/master.blade.php, utilice el resources/views/layouts/app.blade.php.

Borra el ficheroresources/views/layouts/master.blade.php.

Para cambiar el título de la página y el que aparece en el navbar, lo mejor es configurar el nombre de la aplicación. Esto se puede hacer en 2 ficheros: .env o config/app.php. En nuestro caso, lo haremos en .env, sabiendo que si alguien se descarga nuestra aplicación, tendrá que volver a configurar ese nombre de la aplicación. Si lo hubiéramos configurado en config/app.php, aquel que se descargue nuestra aplicación, ya tendrá configurado el nombre.

Comprueba en este punto que el sistema de autenticación funciona correctamente: no te permite entrar a la rutas protegidas si no estás autenticado, puedes acceder con los usuarios definidos en el fichero de semillas y funciona el botón de cerrar sesión.
Ejercicio 5 - Añadir y editar películas (1 punto)

En primer lugar vamos a añadir las rutas que nos van a hacer falta para recoger los datos al enviar los formularios. Para esto editamos el fichero de rutas y añadimos dos rutas (también protegidas por el filtro auth):

    Una ruta de tipo POST para la url catalog/create que apuntará al método postCreate del controlador CatalogController.
    Y otra ruta tipo PUT para la url catalog/edit que apuntará al método putEdit del controlador CatalogController.

A continuación vamos a editar la vista catalog/edit.blade.php con los siguientes cambios:

    Revisar que el método de envío del formulario sea tipo PUT.
    Tenemos que modificar todos los inputs para que, como valor del campo, ponga el valor correspondiente de la película. Por ejemplo en el primer input tendríamos que añadir value="{{$pelicula->title}}". Realiza lo mismo para el resto de campos: year, director, poster y synopsis. El único campo distinto será el de synopsis ya que el input es tipo textarea, en este caso el valor lo tendremos que poner directamente entre la etiqueta de apertura y la de cierre.

Por último tenemos que actualizar el controlador CatalogController con los dos nuevos métodos. En ambos casos tenemos que usar la inyección de dependencias para añadir la clase Request como parámetro de entrada (revisa la sección "Datos de entrada" de la teoría). Además para cada método haremos:

    En el método postCreate creamos una nueva instancia del modelo Movie, asignamos el valor de todos los campos de entrada (title, year, director, poster y synopsis) y los guardamos. Por último, después de guardar, hacemos una redirección a la ruta /catalog.
    En el método putEdit buscamos la película con el identificador pasado por parámetro, actualizamos sus campos y los guardamos. Por último realizamos una redirección a la pantalla con la vista detalle de la película editada.

    Nota: de momento en caso de error no se mostrará nada.

