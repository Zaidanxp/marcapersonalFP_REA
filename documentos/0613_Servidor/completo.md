5. Datos de entrada y Control de usuarios

En este cuarto capítulo vamos a aprender como recoger los datos de entrada de formularios o de algún otro tipo de petición (como por ejemplo una petición de una API). También veremos como leer ficheros de entrada.

En la sección de control de usuarios se tratará todo lo referente a la gestión de los usuarios de una aplicación web, desde como crear la tabla de usuarios, como registrarlos, autenticarlos en la aplicación, cerrar la sesión o como proteger las partes privadas de nuestra aplicación de accesos no permitidos.
5.1. Datos de entrada
Laravel facilita el acceso a los datos de entrada del usuario a través de solo unos pocos métodos. No importa el tipo de petición que se haya realizado (POST, GET, PUT, DELETE), si los datos son de un formulario o si se han añadido a la query string, en todos los casos se obtendrán de la misma forma.

Para conseguir acceso a estos métodos Laravel utiliza inyección de dependencias. Esto es simplemente añadir la clase Request al constructor o método del controlador en el que lo necesitemos. Laravel se encargará de inyectar dicha dependencia ya inicializada y directamente podremos usar este parámetro para obtener los datos de entrada. A continuación se incluye un ejemplo:

<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Routing\Controller;

class UserController extends Controller
{
    public function store(Request $request)
    {
        $name = $request->input('nombre');

        //...
    }
}

En este ejemplo como se puede ver se ha añadido la clase Request como parámetro al método store. Laravel automáticamente se encarga de inyectar estas dependencias por lo que directamente podemos usar la variable $request para obtener los datos de entrada.

Si el método del controlador tuviera más parámetros simplemente los tendremos que añadir a continuación de las dependencias, por ejemplo:

public function edit(Request $request, $id)
{
    //...
}

A continuación veremos los métodos y datos que podemos obtener a partir de la variable $request.
Obtener los valores de entrada

Para obtener el valor de una variable de entrada usamos el método input indicando el nombre de la variable:

$name = $request->input('nombre');

// O simplemente....
$name = $request->nombre;

También podemos especificar un valor por defecto como segundo parámetro:

$name = $request->input('nombre', 'Pedro');

Comprobar si una variable existe

Si lo necesitamos podemos comprobar si un determinado valor existe en los datos de entrada:

if ($request->has('nombre'))
{
    //...
}

Obtener datos agrupados

O también podemos obtener todos los datos de entrada a la vez (en un array) o solo algunos de ellos:

// Obtener todos: 
$input = $request->all();

// Obtener solo los campos indicados: 
$input = $request->only('username', 'password');

// Obtener todos excepto los indicados: 
$input = $request->except('credit_card');

Obtener datos de un array

Si la entrada proviene de un input tipo array de un formulario (por ejemplo una lista de checkbox), si queremos podremos utilizar la siguiente notación con puntos para acceder a los elementos del array de entrada:

$input = $request->input('products.0.name');

JSON

Si la entrada está codificada formato JSON (por ejemplo cuando nos comunicamos a través de una API es bastante común) también podremos acceder a los diferentes campos de los datos de entrada de forma normal (con los métodos que hemos visto, por ejemplo: $nombre = $request->input('nombre');).
Ficheros de entrada

Laravel facilita una serie de clases para trabajar con los ficheros de entrada. Por ejemplo para obtener un fichero que se ha enviado en el campo con nombre photo y guardarlo en una variable, tenemos que hacer:

$file = $request->file('photo');

// O simplemente...
$file = $request->photo;

Si queremos podemos comprobar si un determinado campo tiene un fichero asignado:

if ($request->hasFile('photo')) {
    //...
}

El objeto que recuperamos con $request->file() es una instancia de la clase Symfony\Component\HttpFoundation\File\UploadedFile, la cual extiende la clase de PHP SplFileInfo (http://php.net/manual/es/class.splfileinfo.php), por lo tanto, tendremos muchos métodos que podemos utilizar para obtener datos del fichero o para gestionarlo.

Por ejemplo, para comprobar si el fichero que se ha subido es válido:

if ($request->file('photo')->isValid()) {
    //...
}

En la última versión de Laravel se ha incorporado una nueva librería que nos permite gestionar el acceso y escritura de ficheros en un almacenamiento. Lo interesante de esto es que nos permite manejar de la misma forma el almacenamiento en local, en Amazon S3 y en Rackspace Cloud Storage, simplemente lo tenemos que configurar en config/filesystems.php y posteriormente los podremos usar de la misma forma. Por ejemplo, para almacenar un fichero subido mediante un formulario tenemos que usar el método store indicando como parámetro la ruta donde queremos almacenar el fichero (sin el nombre del fichero):

$path = $request->photo->store('images');
$path = $request->photo->store('images', 's3');  // Especificar un almacenamiento

Estos métodos devolverán el path hasta el fichero almacenado de forma relativa a la raíz de disco configurada. Para el nombre del fichero se generará automáticamente un UUID (identificador único universal). Si queremos especificar nosotros el nombre tendríamos que usar el método storeAs:

$path = $request->photo->storeAs('images', 'filename.jpg');
$path = $request->photo->storeAs('images', 'filename.jpg', 's3');

Otros métodos que podemos utilizar para recuperar información del fichero son:

// Obtener la ruta:
$path = $request->file('photo')->getRealPath();

// Obtener el nombre original:
$name = $request->file('photo')->getClientOriginalName();

// Obtener la extensión: 
$extension = $request->file('photo')->getClientOriginalExtension();

// Obtener el tamaño: 
$size = $request->file('photo')->getSize();

// Obtener el MIME Type:
$mime = $request->file('photo')->getMimeType();

5.2. Control de usuarios

Laravel incluye una serie de métodos y clases que harán que la implementación del control de usuarios sea muy rápida y sencilla. De hecho, casi todo el trabajo ya está hecho, solo tendremos que indicar donde queremos utilizarlo y algunos pequeños detalles de configuración.

Por defecto, al crear un nuevo proyecto de Laravel, ya se incluye todo lo necesario:

    La configuración predeterminada en config/auth.php.
    La migración para la base de datos de la tabla de usuarios con todos los campos necesarios.
    El modelo de datos de usuario (User.php) dentro de la carpeta app con toda la implementación necesaria.
    Los controladores para gestionar todas las acciones relacionadas con el control de usuarios (dentro de App\Http\Controllers\Auth).

Además de esto tendremos que ejecutar los siguientes comandos para generar las rutas y vistas necesarias para realizar el login, registro y para recuperar la contraseña con Laravel/Breeze.

composer update
composer require laravel/breeze:1.9.4 --dev

php artisan breeze:install

npm install
npm run dev
php artisan migrate

En los siguientes apartados vamos a ver en detalle cada uno de estos puntos, desde la configuración hasta los módulos, rutas y vistas por los que está compuesto. En las últimas secciones revisaremos también cómo utilizar este sistema para proteger nuestro sitio web.
Configuración inicial

La configuración del sistema de autenticación se puede encontrar en el fichero config/auth.php, el cual contiene varias opciones (bien documentadas) que nos permitirán, por ejemplo: cambiar el sistema de autenticación (que por defecto es a través de Eloquent), cambiar el modelo de datos usado para los usuarios (por defecto será User) y cambiar la tabla de usuarios (que por defecto será users). Si vamos a utilizar estos valores no será necesario que realicemos ningún cambio.

La migración de la tabla de usuarios (llamada users) también está incluida (ver carpeta database/migrations). Por defecto incluye todos los campos necesarios (ver el código siguiente), pero si necesitamos alguno más lo podemos añadir para guardar por ejemplo la dirección o el teléfono del usuario. A continuación se incluye el código de la función up de la migración:

Schema::create('users', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
    $table->string('email')->unique();
    $table->string('password');
    $table->rememberToken();
    $table->timestamps();
});

Como se puede ver el nombre de la tabla es users, con un índice id autoincremental, y los campos de name, email, password, donde el campo email se establece como único para que no se puedan almacenar emails repetidos. Además se añaden los timestamps que usa Eloquent para almacenar automáticamente la fecha de registro y actualización, y el campo remember_token para recordar la sesión del usuario.

En la carpeta app se encuentra el modelo de datos (llamado User.php) para trabajar con los usuarios. Esta clase ya incluye toda la implementación necesaria y por defecto no tendremos que modificar nada. Pero si queremos podemos modificar esta clase para añadirle más métodos o relaciones con otras tablas, etc.

Laravel también incluye varios controladores (LoginController, RegisterController, ResetPasswordController y ForgotPasswordController) para la autenticación de usuarios, los cuales los puedes encontrar en el espacio de nombres App\Http\Controllers\Auth (y en la misma carpeta). LoginController y RegisterController incluyen métodos para ayudarnos en el proceso de autenticación, registro y cierre de sesión; mientras que ResetPasswordController y ForgotPasswordController contienen la lógica para ayudarnos en el proceso de restaurar una contraseña. Para la mayoría de aplicaciones con estos métodos será suficiente y no tendremos que añadir nada más.
Rutas

Por defecto Laravel no incluye las rutas para el control de usuarios. No obstante, al instalar Laravel/Breeze podemos observar que se añade una línea en el fichero routes/web.php:

require __DIR__.'/auth.php';

Esa línea incluye todas las rutas definidas en el fichero routes/auth.php, las cuales puedes comprobar con el comando


php artisan route:list

Como se puede ver estas rutas ya están enlazadas con los controladores y métodos que incorpora el propio Laravel.
Como la instalación de Laravel/Breeze ha regenerado el fichero routes/web.php:, debemos regresar a la versión anterior, añadiendo tan solo la línea


require __DIR__.'/auth.php';

Para ello, copiaremos la línea anterior y desde un terminal ejecutamos el siguiente comando git desde el directorio de videoclub:

Vistas

Al instalar Laravel/Breeze, también se generarán todas las vistas necesarias para realizar el login, registro y para recuperar la contraseña. Todas estas vistas las podremos encontrar en la carpeta resources/views/auth con los nombres login.blade.php para el formulario de login, register.blade.php , etcétera. Estos nombres y rutas son obligatorios ya que los controladores que incluye Laravel accederán a ellos, por lo que no deberemos cambiarlos.

Autenticación de un usuario

Una vez configurado todo el sistema, añadidas las rutas y las vistas para realizar el control de usuarios ya podemos utilizarlo. Si accedemos a la ruta login nos aparecerá la vista con el formulario de login, solicitando nuestro email y contraseña para acceder. El campo tipo checkbox llamado "remember" nos permitirá indicar si deseamos que la sesión permanezca abierta hasta que se cierre manualmente. Es decir, aunque se cierre el navegador y pasen varios días el usuario seguiría estando autorizado.

Si los datos introducidos son correctos se creará la sesión del usuario y se le redirigirá a la ruta "/dashboard". Si queremos cambiar esta ruta tenemos que definir la constante HOME en el fichero app/Providers/RouteServiceProvider.php, por ejemplo:

    public const HOME = '/catalog';

Registro de un usuario

Si accedemos a la ruta register nos aparecerá la vista con el formulario de registro, solicitándonos los campos nombre, email y contraseña. Al pulsar el botón de envío del formulario se llamará a la ruta register por POST y se almacenará el nuevo usuario en la base de datos.

Si no hemos añadido ningún campo más en la migración no tendremos que configurar nada más. Sin embargo si hemos añadido algún campo más a la tabla de usuarios tendremos que actualizar el controlador RegisteredUserController: validate y create. En la llamada a validate simplemente tendremos que añadir dicho campo al array de validaciones (solo en el caso que necesitemos validarlo). Y en la llamada al método create tendremos que añadir los campos adicionales que deseemos almacenar. El código de este método es el siguiente:

protected function create(array $data) {
    return User::create([
        'name' => $data['name'],
        'email' => $data['email'],
        'phone' => $data['phone'],     // Campo añadido
        'password' => bcrypt($data['password']),
    ]);
}

Como podemos ver utiliza el modelo de datos User para crear el usuario y almacenar las variables que recibe en el array de datos $request. En este array de datos nos llegarán todos los valores de los campos del formulario, por lo tanto, si añadimos más campos al formulario y a la tabla de usuarios simplemente tendremos que añadirlos también en este método.

Es importante destacar que la contraseña se cifra usando el método bcrypt, por lo tanto las contraseñas se almacenaran cifradas en la base de datos. Este cifrado se basa en la clave hash que se general al crear un nuevo proyecto de Laravel (ver capítulo de "Instalación") y que se encuentra almacenada en el fichero .env en la variable APP_KEY. Es importante que este hash se haya establecido al inicio (que no esté vacío o se uno por defecto) y que además no se modifique una vez la aplicación se suba a producción.
Registro manual de un usuario

Si queremos añadir un usuario manualmente lo podemos hacer de forma normal usando el modelo User de Eloquent, con la única precaución de cifrar la contraseña que se va a almacenar. A continuación se incluye un ejemplo de una función que crea un nuevo usuario a partir de los parámetros de entrada recibidos de un formulario:

public function store(Request $request) {
    $user = new User;
    $user->name = $request->input('name');
    $user->email = $request->input('email');
    $user->password = bcrypt( $request->input('password') );
    $user->save();
}

Acceder a los datos del usuario autenticado

Una vez que el usuario está autenticado podemos acceder a los datos del mismo a través del método Auth::user(), por ejemplo:

$user = Auth::user();

Este método nos devolverá null en caso de que no esté autenticado. Si estamos seguros de que el usuario está autenticado (porque estamos en una ruta protegida) podremos acceder directamente a sus propiedades:

$email = Auth::user()->email;

    Importante: para utilizar la clase Auth tenemos que añadir el espacio de nombres use Illuminate\Support\Facades\Auth;, de otra forma nos aparecerá un error indicando que no puede encontrar la clase.

El usuario también se inyecta en los parámetros de entrada de la petición (en la clase Request). Por lo tanto, si en un método de un controlador usamos la inyección de dependencias también podremos acceder a los datos del usuario:

use Illuminate\Http\Request;

class ProfileController extends Controller {
    public function updateProfile(Request $request) {
        if ($request->user()) {
            $email = $request->user()->email;
        }
    }
}

Cerrar la sesión

Si accedemos a la ruta logout por POST se cerrará la sesión y se redirigirá a la ruta /. Todo esto lo hará automáticamente el método destroy del controlador AuthenticatedSessionController.

Para cerrar manualmente la sesión del usuario actualmente autenticado tenemos que utilizar el método:

Auth::logout();

Posteriormente podremos hacer una redirección a una página principal para usuarios no autenticados.

    Importante: para utilizar la clase Auth tenemos que añadir el espacio de nombres use Illuminate\Support\Facades\Auth;, de otra forma nos aparecerá un error indicando que no puede encontrar la clase.

Comprobar si un usuario está autenticado

Para comprobar si el usuario actual se ha autenticado en la aplicación podemos utilizar el método Auth::check() de la forma:

if (Auth::check()) {
    // El usuario está correctamente autenticado
}

Sin embargo, lo recomendable es utilizar Middleware (como veremos a continuación) para realizar esta comprobación antes de permitir el acceso a determinadas rutas.

    Importante: Recuerda que para utilizar la clase Auth tenemos que añadir el espacio de nombres use Illuminate\Support\Facades\Auth;, de otra forma nos aparecerá un error indicando que no puede encontrar la clase.

Proteger rutas

El sistema de autenticación de Laravel también incorpora una serie de filtros o Middleware (ver carpeta app/Http/Middleware y el fichero app/Http/Kernel.php) para comprobar que el usuario que accede a una determinada ruta o grupo de rutas esté autenticado. En concreto para proteger el acceso a rutas y solo permitir su visualización por usuarios correctamente autenticados usaremos el middleware \Illuminate\Auth\Middleware\Authenticate.php cuyo alias es auth. Para utilizar este middleware tenemos que editar el fichero routes/web.php y modificar las rutas que queramos proteger, por ejemplo:

// Para proteger una clausula:
Route::get('admin/catalog', function() {
    // Solo se permite el acceso a usuarios autenticados
})->middleware('auth');

// Para proteger una acción de un controlador:
Route::get('profile', [ProfileController::class, 'show'])->middleware('auth');

Si el usuario que accede no está validado se generará una excepción que le redirigirá a la ruta login. Si deseamos cambiar esta dirección tendremos que modificar el método que gestiona la excepción, el cual lo podremos encontrar en App\Exceptions\Handler@unauthenticated.

Si deseamos proteger el acceso a toda una zona de nuestro sitio web (por ejemplo la parte de administración o la gestión de un recurso), lo más cómodo es crear un grupo con todas esas rutas que utilice el middleware auth, por ejemplo:

Route::group(['middleware' => 'auth'], function() {
    Route::get('catalog', [CatalogController::class, 'getIndex']);
    Route::get('catalog/create', [CatalogController::class, 'getCreate']);
    // ...
});

5.3. Ejercicios

En los ejercicios de esta sección vamos a completar el proyecto del videoclub terminando el procesamiento de los formularios y añadiendo el sistema de autenticación de usuarios.
Ejercicio 1 - Migración de la tabla usuarios (0.5 puntos)

En primer lugar vamos a crear la tabla de la base de datos para almacenar los usuarios que tendrán acceso a la plataforma de gestión del videoclub.

Como hemos visto en la teoría, Laravel ya incluye una migración con el nombre create_users_table para la tabla users con todos los campos necesarios. Vamos a abrir esta migración y a comprobar que los campos incluidos coinciden con los de la siguiente tabla:
Campo	Tipo	Modificador
id 	Autoincremental 	
name 	String 	
email 	String 	unique
password 	String 	
remember_token 	Campo remember_token 	
timestamps 	Timestamps de Eloquent 	 

Comprueba también que en el método down de la migración se deshagan los cambios que se hacen en el método up, en este caso sería eliminar la tabla.

Por último usamos el comando de Artisan que añade las nuevas migraciones y comprobamos con PHPMyAdmin que la tabla se ha creado correctamente con todos campos indicados.
Ejercicio 2 - Seeder de usuarios (0.5 puntos)

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

