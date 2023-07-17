3. Controladores, filtros y formularios

En este capítulo empezaremos a utilizar realmente el patrón MVC y veremos como añadir controladores a nuestros proyectos web. Además, también veremos el concepto de filtros mediante Middleware aplicados sobre el fichero de rutas y como realizar redirecciones en Laravel. Por último se incluye una sección de ejercicios para practicar con todo lo aprendido.
3.1. Controladores

Hasta el momento hemos visto solamente como devolver una cadena para una ruta y como asociar una vista a una ruta directamente en el fichero de rutas. Pero en general la forma recomendable de trabajar será asociar dichas rutas a un método de un controlador. Esto nos permitirá separar mucho mejor el código y crear clases (controladores) que agrupen toda la funcionalidad de un determinado recurso. Por ejemplo, podemos crear un controlador para gestionar toda la lógica asociada al control de usuarios o cualquier otro tipo de recurso.

Como ya vimos en la sección de introducción, los controladores son el punto de entrada de las peticiones de los usuarios y son los que deben contener toda la lógica asociada al procesamiento de una petición, encargándose de realizar las consultas necesarias a la base de datos, de preparar los datos y de llamar a la vista correspondiente con dichos datos.

Controlador básico

Los controladores se almacenan en ficheros PHP en la carpeta app/Http/Controllers y normalmente se les añade el sufijo Controller, por ejemplo UserController.php o MoviesController.php. A continuación se incluye un ejemplo básico de un controlador almacenado en el fichero app/Http/Controllers/UserController.php:

<?php
namespace App\Http\Controllers;

use App\User;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * Mostrar información de un usuario.
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        $user = User::findOrFail($id);
        return view('user.profile', ['user' => $user]);
    }
}

Todos los controladores tienen que extender la clase base Controller. Esta clase viene ya creada por defecto con la instalación de Laravel, la podemos encontrar en la carpeta app/Http/Controllers. Se utiliza para centralizar toda la lógica que se vaya a utilizar de forma compartida por los controladores de nuestra aplicación. Por defecto solo carga código para validación y autorización, pero podemos añadir en la misma todos los métodos que necesitemos.

En el código de ejemplo, el método showProfile($id) lo único que realiza es obtener los datos de un usuario, generar la vista user.profile a partir de los datos obtenidos y devolverla como valor de retorno para que se muestre por pantalla.

Una vez definido un controlador ya podemos asociarlo a una ruta. Para esto tenemos que modificar el fichero de rutas routes.php de la forma:

use App\Http\Controllers\UserController;
Route::get('user/{id}', [UserController::class, 'showProfile']);

En lugar de pasar una función como segundo parámetro, tenemos que escribir una cadena que contenga el nombre del controlador, seguido de una arroba @ y del nombre del método que queremos asociar. No es necesario añadir nada más, ni los parámetros que recibe el método en cuestión, todo esto se hace de forma automática.

Crear un nuevo controlador

Como hemos visto los controladores se almacenan dentro de la carpeta app/Http/Controllers como ficheros PHP. Para crear uno nuevo bien lo podemos hacer a mano y rellenar nosotros todo el código, o podemos utilizar el siguiente comando de Artisan que nos adelantará todo el trabajo:

php artisan make:controller MoviesController

Este comando creará el controlador MoviesController dentro de la carpeta app/Http/Controllers y lo completará con el código básico que hemos visto antes.

Controladores y espacios de nombres

También podemos crear sub-carpetas dentro de la carpeta Controllers para organizarnos mejor. En este caso, la estructura de carpetas que creemos no tendrá nada que ver con la ruta asociada a la petición y, de hecho, a la hora de hacer referencia al controlador únicamente tendremos que hacerlo a través de su espacio de nombres.

Como hemos visto al referenciar el controlador en el fichero de rutas únicamente tenemos que indicar su nombre y no toda la ruta ni el espacio de nombres App\Http\Controllers. Esto es porque el servicio encargado de cargar las rutas añade automáticamente el espacio de nombres raíz para los controladores. Si metemos todos nuestros controladores dentro del mismo espacio de nombres no tendremos que añadir nada más. Pero si decidimos crear sub-carpetas y organizar nuestros controladores en sub-espacios de nombres, entonces sí que tendremos que añadir esa parte.

Por ejemplo, si creamos un controlador en App\Http\Controllers\Photos\AdminController, entonces para registrar una ruta hasta dicho controlador tendríamos que hacer:

Route::get('foo', 'Photos\AdminController@method');

Generar una URL a una acción

Para generar la URL que apunte a una acción de un controlador podemos usar el método action de la forma:

$url = action('FooController@method');

Por ejemplo, para crear en una plantilla con Blade un enlace que apunte a una acción haríamos:

<a href="{{ action('FooController@method') }}">¡Aprieta aquí!</a>

Caché de rutas

Si definimos todas nuestras rutas para que utilicen controladores podemos aprovechar la nueva funcionalidad para crear una caché de las rutas. Es importante que estén basadas en controladores porque si definimos respuestas directas desde el fichero de rutas (como vimos en el capítulo anterior) la caché no funcionará.

Gracias a la caché Laravel indican que se puede acelerar el proceso de registro de rutas hasta 100 veces. Para generar la caché simplemente tenemos que ejecutar el comando de Artisan:

php artisan route:cache

Si creamos más rutas y queremos añadirlas a la caché simplemente tenemos que volver a lanzar el mismo comando. Para borrar la caché de rutas y no generar una nueva caché tenemos que ejecutar:

php artisan route:clear

La caché se recomienda crearla solo cuando ya vayamos a pasar a producción nuestra web. Cuando estamos trabajando en la web es posible que añadamos nuevas rutas y sino nos acordamos de regenerar la caché la ruta no funcionará.
3.2. Middleware o filtros

Los componentes llamados Middleware son un mecanismo proporcionado por Laravel para filtrar las peticiones HTTP que se realizan a una aplicación. Un filtro o middleware se define como una clase PHP almacenada en un fichero dentro de la carpeta app/Http/Middleware. Cada middleware se encargará de aplicar un tipo concreto de filtro y de decidir que realizar con la petición realizada: permitir su ejecución, dar un error o redireccionar a otra página en caso de no permitirla.

Laravel incluye varios filtros por defecto, uno de ellos es el encargado de realizar la autenticación de los usuarios. Este filtro lo podemos aplicar sobre una ruta, un conjunto de rutas o sobre un controlador en concreto. Este middleware se encargará de filtrar las peticiones a dichas rutas: en caso de estar logueado y tener permisos de acceso le permitirá continuar con la petición, y en caso de no estar autenticado lo redireccionará al formulario de login.

Laravel incluye middleware para gestionar la autenticación, el modo mantenimiento, la protección contra CSRF, y algunos mas. Todos estos filtros los podemos encontrar en la carpeta app/Http/Middleware, los cuales los podemos modificar o ampliar su funcionalidad. Pero además de estos podemos crear nuestros propios Middleware como veremos a continuación.
Definir un nuevo Middleware

Para crear un nuevo Middleware podemos utilizar el comando de Artisan:

php artisan make:middleware MyMiddleware

Este comando creará la clase MyMiddleware dentro de la carpeta app/Http/Middleware con el siguiente contenido por defecto:

<?php

namespace App\Http\Middleware;

use Closure;

class MyMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        return $next($request);
    }
}

El código generado por Artisan ya viene preparado para que podamos escribir directamente la implementación del filtro a realizar dentro de la función handle. Como podemos ver, esta función solo incluye el valor de retorno con una llamada a return $next($request);, que lo que hace es continuar con la petición y ejecutar el método que tiene que procesarla. Como entrada la función handle recibe dos parámetros:

    $request: En la cual nos vienen todos los parámetros de entrada de la peticion.
    $next: El método o función que tiene que procesar la petición.

Por ejemplo podríamos crear un filtro que redirija al home si el usuario tiene menos de 18 años y en otro caso que le permita acceder a la ruta:

public function handle($request, Closure $next)
{
    if ($request->input('age') < 18) {
        return redirect('home');
    }

    return $next($request);
}

Como hemos dicho antes, podemos hacer tres cosas con una petición:

    Si todo es correcto permitir que la petición continúe devolviendo: return $next($request);
    Realizar una redirección a otra ruta para no permitir el acceso con: return redirect('home');
    Lanzar una excepción o llamar al método abort para mostrar una página de error: abort(403, 'Unauthorized action.');

Middleware antes o después de la petición

Para hacer que el código de un Middleware se ejecute antes o después de la petición HTTP simplemente tenemos que poner nuestro código antes o después de la llamada a $next($request);. Por ejemplo, el siguiente _Middleware realizaría la acción antes de la petición:

public function handle($request, Closure $next)
{
    // Código a ejecutar antes de la petición

    return $next($request);
}

Mientras que el siguiente Middleware ejecutaría el código después de la petición:

public function handle($request, Closure $next)
{
    $response = $next($request);

    // Código a ejecutar después de la petición

    return $response;
}

Uso de Middleware

De momento hemos visto para que vale y como se define un Middleware, en esta sección veremos como utilizarlos. Laravel permite la utilización de Middleware de tres formas distintas: global, asociado a rutas o grupos de rutas, o asociado a un controlador o a un método de un controlador. En los tres casos será necesario registrar primero el Middleware en la clase app/Http/Kernel.php.
Middleware global

Para hacer que un Middleware se ejecute con todas las peticiones HTTP realizadas a una aplicación simplemente lo tenemos que registrar en el array $middleware definido en la clase app/Http/Kernel.php. Por ejemplo:

protected $middleware = [
    \Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode::class,
    \App\Http\Middleware\EncryptCookies::class,
    \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
    \Illuminate\Session\Middleware\StartSession::class,
    \Illuminate\View\Middleware\ShareErrorsFromSession::class,
    \App\Http\Middleware\VerifyCsrfToken::class,
    \App\Http\Middleware\MyMiddleware::class,
];

En este ejemplo hemos registrado la clase MyMiddleware al final del array. Si queremos que nuestro middleware se ejecute antes que otro filtro simplemente tendremos que colocarlo antes en la posición del array.
Middleware asociado a rutas

En el caso de querer que nuestro middleware se ejecute solo cuando se llame a una ruta o a un grupo de rutas también tendremos que registrarlo en el fichero app/Http/Kernel.php, pero en el array $routeMiddleware. Al añadirlo a este array además tendremos que asignarle un nombre o clave, que será el que después utilizaremos asociarlo con una ruta.

En primer lugar añadimos nuestro filtro al array y le asignamos el nombre "es_mayor_de_edad":

protected $routeMiddleware = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    'es_mayor_de_edad' => \App\Http\Middleware\MyMiddleware::class,
];

Una vez registrado nuestro middleware ya lo podemos utilizar en el fichero de rutas app/Http/routes.php mediante la clave o nombre asignado, por ejemplo:

Route::get('dashboard', ['middleware' => 'es_mayor_de_edad', function () {
    //...
}]);

En el ejemplo anterior hemos asignado el middleware con clave es_mayor_de_edad a la ruta dashboard. Como se puede ver se utiliza un array como segundo parámetro, en el cual indicamos el middleware y la acción. Si la petición supera el filtro entonces se ejecutara la función asociada.

Para asociar un filtro con una ruta que utiliza un método de un controlador se realizaría de la misma manera pero indicando la acción mediante la clave "uses":

Route::get('profile', [
    'middleware' => 'auth',
    'uses' => 'UserController@showProfile'
]);

Si queremos asociar varios middleware con una ruta simplemente tenemos que añadir un array con las claves. Los filtros se ejecutarán en el orden indicado en dicho array:

Route::get('dashboard', ['middleware' => ['auth', 'es_mayor_de_edad'], function () {
    //...
}]);

Laravel también permite asociar los filtros con las rutas usando el método middleware() sobre la definición de la ruta de la forma:

Route::get('/', function () {
    // ...
})->middleware(['first', 'second']);

// O sobre un controlador: 
Route::get('profile', 'UserController@showProfile')->middleware('auth');

Middleware dentro de controladores

También es posible indicar el middleware a utilizar desde dentro de un controlador. En este caso los filtros también tendrán que estar registrador en el array $routeMiddleware del fichero app/Http/Kernel.php. Para utilizarlos se recomienda realizar la asignación en el constructor del controlador y asignar los filtros usando su clave mediante el método middleware. Podremos indicar que se filtren todos los métodos, solo algunos, o todos excepto los indicados, por ejemplo:

class UserController extends Controller
{
    /**
     * Instantiate a new UserController instance.
     *
     * @return void
     */
    public function __construct()
    {
        // Filtrar todos los métodos
        $this->middleware('auth');

        // Filtrar solo estos métodos...
        $this->middleware('log', ['only' => ['fooAction', 'barAction']]);

        // Filtrar todos los métodos excepto...
        $this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
    }
}

Revisar los filtros asignados

Al crear una aplicación Web es importante asegurarse de que todas las rutas definidas son correctas y que las partes privadas realmente están protegidas. Para esto Laravel incluye el siguiente método de Artisan:

php artisan route:list

Este método muestra una tabla con todas las rutas, métodos y acciones. Ademas para cada ruta indica los filtros asociados, tanto si están definidos desde el fichero de rutas como desde dentro de un controlador. Por lo tanto es muy útil para comprobar que todas las rutas y filtros que hemos definido se hayan creado correctamente.
Paso de parámetros

Un Middleware también puede recibir parámetros. Por ejemplo, podemos crear un filtro para comprobar si el usuario logueado tiene un determinado rol indicado por parámetro. Para esto lo primero que tenemos que hacer es añadir un tercer parámetro a la función handle del Middleware:

<?php

namespace App\Http\Middleware;

use Closure;

class RoleMiddleware
{
    /**
     * Run the request filter.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @param  string  $role
     * @return mixed
     */
    public function handle($request, Closure $next, $role)
    {
        if (! $request->user()->hasRole($role)) {
            // No tiene el rol esperado!
        }

        return $next($request);
    }

}

En el código anterior de ejemplo se ha añadido el tercer parámetro $role a la función. Si nuestro filtro necesita recibir más parámetros simplemente tendríamos que añadirlos de la misma forma a esta función.

Para pasar un parámetro a un middleware en la definición de una ruta lo tendremos que añadir a continuación del nombre del filtro separado por dos puntos, por ejemplo:

Route::put('post/{id}', ['middleware' => 'role:editor', function ($id) {
    //
}]);

Si tenemos que pasar más de un parámetro al filtro los separaremos por comas, por ejemplo: role:editor,admin.
3.3. Rutas avanzadas

Laravel permite crear grupos de rutas para especificar opciones comunes a todas ellas, como por ejemplo un middleware, un prefijo, un subdominio o un espacio de nombres que se tiene que aplicar sobre todas ellas.

A continuación vamos a ver algunas de estas opciones, en todos los casos usaremos el método Route::group, el cual recibirá como primer parámetro las opciones a aplicar sobre todo el grupo y como segundo parámetro una clausula con la definición de las rutas.
Middleware sobre un grupo de rutas

Esta opción es muy útil para aplicar un filtro sobre todo un conjunto de rutas, de esta forma solo tendremos que especificar el filtro una vez y además nos permitirá dividir las rutas en secciones (distinguiendo mejor a que secciones se les está aplicando un filtro):

Route::group(['middleware' => 'auth'], function () {
    Route::get('/', function ()    {
        // Ruta filtrada por el middleware
    });

    Route::get('user/profile', function () {
        // Ruta filtrada por el middleware
    });
});

Grupos de rutas con prefijo

También podemos utilizar la opción de agrupar rutas para indicar un prefijo que se añadirá a todas las URL del grupo. Por ejemplo, si queremos definir una sección de rutas que empiecen por el prefijo dashboard tendríamos que hacer lo siguiente:

Route::group(['prefix' => 'dashboard'], function () {
    Route::get('catalog', function () { /* ... */ });
    Route::get('users', function () { /* ... */ });
});

También podemos crear grupos de rutas dentro de otros grupos. Por ejemplo para definir un grupo de rutas a utilizar en una API y crear diferentes rutas según la versión de la API podríamos hacer:

Route::group(['prefix' => 'api'], function()
{
    Route::group(['prefix' => 'v1'], function()
    {
        // Rutas con el prefijo api/v1
        Route::get('recurso',      'ControllerAPIv1@getRecurso');
        Route::post('recurso',     'ControllerAPIv1@postRecurso');
        Route::get('recurso/{id}', 'ControllerAPIv1@putRecurso');
    });

    Route::group(['prefix' => 'v2'], function()
    {
        // Rutas con el prefijo api/v2
        Route::get('recurso',      'ControllerAPIv2@getRecurso');
        Route::post('recurso',     'ControllerAPIv2@postRecurso');
        Route::get('recurso/{id}', 'ControllerAPIv2@putRecurso');
    });
});

De esta forma podemos crear secciones dentro de nuestro fichero de rutas para agrupar, por ejemplo, todas las rutas públicas, todas las de la sección privada de administración, sección privada de usuario, las rutas de las diferentes versiones de la API de nuestro sitio, etc.

Esta opción también la podemos aprovechar para especificar parámetros comunes que se recogerán para todas las rutas y se pasarán a todos los controladores o funciones asociadas, por ejemplo:

Route::group(['prefix' => 'accounts/{account_id}'], function () {
    Route::get('detail', function ($account_id)  { /* ... */ });
    Route::get('settings', function ($account_id)  { /* ... */ });
});

3.4. Redirecciones

Como respuesta a una petición también podemos devolver una redirección. Esta opción será interesante cuando, por ejemplo, el usuario no esté logueado y lo queramos redirigir al formulario de login, o cuando se produzca un error en la validación de una petición y queramos redirigir a otra ruta.

Para esto simplemente tenemos que utilizar el método redirect indicando como parámetro la ruta a redireccionar, por ejemplo:

return redirect('user/login');

O si queremos volver a la ruta anterior simplemente podemos usar el método back:

return back();

Redirección a una acción de un controlador

También podemos redirigir a un método de un controlador mediante el método action de la forma:

return redirect()->action('HomeController@index');

Si queremos añadir parámetros para la llamada al método del controlador tenemos que añadirlos pasando un array como segundo parámetro:

return redirect()->action('UserController@profile', [1]);

Redirección con los valores de la petición

Las redirecciones se suelen utilizar tras obtener algún error en la validación de un formulario o tras procesar algunos parámetros de entrada. En este caso, para que al mostrar el formulario con los errores producidos podamos añadir los datos que había escrito el usuario tendremos que volver a enviar los valores enviados con la petición usando el método withInput():

return redirect('form')->withInput();

// O para reenviar los datos de entrada excepto algunos:
return redirect('form')->withInput($request->except('password'));

Este método también lo podemos usar con la función back o con la función action:

return back()->withInput();

return redirect()->action('HomeController@index')->withInput();

3.5. Formularios

La última versión de Laravel no incluye ninguna utilidad para la generación de formularios. En esta sección vamos a repasar brevemente como crear un formulario usando etiquetas de HTML, los distintos elementos o inputs que podemos utilizar, además también veremos como conectar el envío de un formulario con un controlador, como protegernos de ataques CSRF y algunas cuestiones más.
Crear formularios

Para abrir y cerrar un formulario que apunte a la URL actual y utilice el método POST tenemos que usar las siguientes etiquetas HTML:

<form method="POST">
    ...
</form>

Si queremos cambiar la URL de envío de datos podemos utilizar el atributo action de la forma:

<form action="{{ url('foo/bar') }}" method="POST">
    ...
</form>

La función url generará la dirección a la ruta indicada. Ademas también podemos usar la función action para indicar directamente el método de un controlador a utilizar, por ejemplo: action([HomeController::class, 'getIndex'])

Como hemos visto anteriormente, en Laravel podemos definir distintas acciones para procesar peticiones realizadas a una misma ruta pero usando un método distinto (GET, POST, PUT, DELETE). Por ejemplo, podemos definir la ruta "user" de tipo GET para que nos devuelva la página con el formulario para crear un usuario, y por otro lado definir la ruta "user" de tipo POST para procesar el envío del formulario. De esta forma cada ruta apuntará a un método distinto de un controlador y nos facilitará la separación del código.

HTML solo permite el uso de formularios de tipo GET o POST. Si queremos enviar un formulario usando otros de los métodos (o verbos) definidos en el protocolo REST, como son PUT, PATCH o DELETE, tendremos que añadir un campo oculto para indicarlo. Laravel establece el uso del nombre "_method" para indicar el método a usar, por ejemplo:

<form action="/foo/bar" method="POST">
    <input type="hidden" name="_method" value="PUT">
    ...
</form>

Laravel se encargará de recoger el valor de dicho campo y de procesarlo como una petición tipo PUT (o la que indiquemos). Además, para facilitar más la definición de este tipo de formularios ha añadido la función method_field que directamente creará este campo oculto:

<form action="/foo/bar" method="POST">
    {{ method_field('PUT') }}
    ...
</form>

Protección contra CSRF

El CSRF (del inglés Cross-site request forgery o falsificación de petición en sitios cruzados) es un tipo de exploit malicioso de un sitio web en el que comandos no autorizados son transmitidos por un usuario en el cual el sitio web confía.

Laravel proporciona una forma fácil de protegernos de este tipo de ataques. Simplemente tendremos que utilizar la directiva de Blade @csrf después de abrir el formulario, igual que vimos en la sección anterior, este método añadirá un campo oculto ya configurado con los valores necesarios. A continuación se incluye un ejemplo de uso:

<form action="/foo/bar" method="POST">
    @csrf
    ...
</form>

Elementos de un formulario

A continuación vamos a ver los diferentes elementos que podemos añadir a un formulario. En todos los tipos de campos en los que tengamos que recoger datos es importante añadir sus atributos name e id, ya que nos servirán después para recoger los valores rellenados por el usuario.
Campos de texto

Para crear un campo de texto usamos la etiqueta de HTML input, para la cual tenemos que indicar el tipo text y su nombre e identificador de la forma:

<input type="text" name="nombre" id="nombre">

En este ejemplo hemos creado un campo de texto vacío cuyo nombre e identificador es "nombre". El atributo name indica el nombre de variable donde se guardará el texto introducido por el usuario y que después utilizaremos desde el controlador para acceder al valor.

Si queremos podemos especificar un valor por defecto usando el atributo value:

<input type="text" name="nombre" id="nombre" value="Texto inicial">

Desde una vista con Blade podemos asignar el contenido de una variable (en el ejemplo $nombre) para que aparezca el campo de texto con dicho valor. Esta opción es muy útil para crear formularios en los que tenemos que editar un contenido ya existente, como por ejemplo editar los datos de usuario. A continuación se muestra un ejemplo:

<input type="text" name="nombre" id="nombre" value="{{ $nombre }}">

Para mostrar los valores introducidos en una petición anterior podemos usar el método old, el cual recuperará las variables almacenadas en la petición anterior. Por ejemplo, imaginad que creáis un formulario para el registro de usuarios y al enviar el formulario comprobáis que el usuario introducido está repetido. En ese caso se tendría que volver a mostrar el formulario con los datos introducidos y marcar dicho campo como erróneo. Para esto, después de comprobar que hay un error en el controlador, habría que realizar una redirección a la página anterior añadiendo la entrada como ya vimos con withInput(), por ejemplo: return back()->withInput();. El método withInput() añade todas las variables de entrada a la sesión, y esto nos permite recuperarlas después de la forma:

<input type="text" name="nombre" id="nombre" value="{{ old('nombre') }}">

Más adelante, cuando veamos como recoger los datos de entrada revisaremos el proceso completo para procesar un formulario.
Más campos tipo input

Utilizando la etiqueta input podemos crear más tipos de campos como contraseñas o campos ocultos:

<input type="password" name="password" id="password">

<input type="hidden" name="oculto" value="valor">

Los campos para contraseñas lo único que hacen es ocultar las letras escritas. Los campos ocultos se suelen utilizar para almacenar opciones o valores que se desean enviar junto con los datos del formulario pero que no se tienen que mostrar al usuario. En las secciones anteriores ya hemos visto que Laravel lo utiliza internamente para almacenar un hash o código para la protección contra ataques tipo CSRF y que también lo utiliza para indicar si el tipo de envío del formulario es distinto de POST o GET. Además nosotros lo podemos utilizar para almacenar cualquier valor que después queramos recoger justo con los datos del formulario.

También podemos crear otro tipo de inputs como email, number, tel, etc. (podéis consultar la lista de tipos permitidos aquí: http://www.w3schools.com/html/html_form_input_types.asp). Para definir estos campos se hace exactamente igual que para un campo de texto pero cambiando el tipo por el deseado, por ejemplo:

<input type="email" name="correo" id="correo">

<input type="number" name="numero" id="numero">

<input type="tel" name="telefono" id="telefono">

Textarea

Para crear un área de texto simplemente tenemos que usar la etiqueta HTML textarea de la forma:

<textarea name="texto" id="texto"></textarea>

Esta etiqueta además permite indicar el número de filas (rows) y columnas (cols) del área de texto. Para insertar un texto o valor inicial lo tenemos que poner entre la etiqueta de apertura y la de cierre. A continuación se puede ver un ejemplo completo:

<textarea name="texto" id="texto" rows="4" cols="50">Texto por defecto</textarea>

Etiquetas

Las etiquetas nos permiten poner un texto asociado a un campo de un formulario para indicar el tipo de contenido que se espera en dicho campo. Por ejemplo añadir el texto "Nombre" antes de un input tipo texto donde el usuario tendrá que escribir su nombre.

Para crear una etiqueta tenemos que usar el tag "label" de HTML:

<label for="nombre">Nombre</label>

Donde el atributo for se utiliza para especificar el identificador del campo relacionado con la etiqueta. De esta forma, al pulsar sobre la etiqueta se marcará automáticamente el campo relacionado. A continuación se muestra un ejemplo completo:

<label for="correo">Correo electrónico:</label>
<input type="email" name="correo" id="correo">

Checkbox y Radio buttons

Para crear campos tipo checkbox o tipo radio button tenemos que utilizar también la etiqueta input, pero indicando el tipo chekbox o radio respectivamente. Por ejemplo, para crear un checkbox para aceptar los términos escribiríamos:

<label for="terms">Aceptar términos</label>
<input type="checkbox" name="terms" id="terms" value="1">

En este caso, al enviar el formulario, si el usuario marca la casilla nos llegaría la variable con nombre terms con valor 1. En caso de que no marque la casilla no llegaría nada, ni siquiera la variable vacía.

Para crear una lista de checkbox o de radio button es importante que todos tengan el mismo nombre (para la propiedad name). De esta forma los valores devueltos estarán agrupados en esa variable, y además, el radio button funcionará correctamente: al apretar sobre una opción se desmarcará la que este seleccionada en dicho grupo (entre todos los que tengan el mismo nombre). Por ejemplo:

<label for="color">Elige tu color favorito:</label>
<br>
<input type="radio" name="color" id="color" value="rojo">Rojo<br>
<input type="radio" name="color" id="color" value="azul">Azul<br>
<input type="radio" name="color" id="color" value="amarillo">Amarillo<br>
<input type="radio" name="color" id="color" value="verde">Verde<br>

Además podemos añadir el atributo checked para marcar una opción por defecto:

<label for="clase">Clase:</label>
<input type="radio" name="clase" id="clase" value="turista" checked>Turista<br>
<input type="radio" name="clase" id="clase" value="preferente">Preferente<br>

Ficheros

Para generar un campo para subir ficheros utilizamos también la etiqueta input indicando en su tipo el valor file, por ejemplo:

<label for="imagen">Sube la imagen:</label>
<input type="file" name="imagen" id="imagen">

Para enviar ficheros la etiqueta de apertura del formulario tiene que cumplir dos requisitos importantes:

    El método de envío tiene que ser POST o PUT.
    Tenemos que añadir el atributo enctype="multipart/form-data" para indicar la codificacón.

A continuación se incluye un ejemplo completo:

<form enctype="multipart/form-data" method="post">
    <label for="imagen">Sube la imagen:</label>
    <input type="file" name="imagen" id="imagen">
</form>

Listas desplegables

Para crear una lista desplegable utilizamos la etiqueta HTML select. Las opciones la indicaremos entre la etiqueta de apertura y cierre usando elementos option, de la forma:

<select name="marca">
  <option value="volvo">Volvo</option>
  <option value="saab">Saab</option>
  <option value="mercedes">Mercedes</option>
  <option value="audi">Audi</option>
</select>

En el ejemplo anterior se creará una lista desplegable con cuatro opciones. Al enviar el formulario el valor seleccionado nos llegará en la variable marca. Además, para elegir una opción por defecto podemos utilizar el atributo selected, por ejemplo:

<label for="talla">Elige la talla:</label>
<select name="talla" id="talla">
  <option value="XS">XS</option> 
  <option value="S">S</option>
  <option value="M" selected>M</option>
  <option value="L">L</option>
  <option value="XL">XL</option>
</select>

Botones

Por último vamos a ver como añadir botones a un formulario. En un formulario podremos añadir tres tipos distintos de botones:

    submit para enviar el formulario,
    reset para restablecer o borrar los valores introducidos y
    button para crear botones normales para realizar otro tipo de acciones (como volver a la página anterior).

A continuación se incluyen ejemplo de cada uno de ellos:

<button type="submit">Enviar</button>
<button type="reset">Borrar</button>
<button type="button">Volver</button>

Recuperando los datos del Formulario

Para obtener la instancia del objeto Request actual debemos inyectar la clase Illuminate\Http\Request en el método del controlador.

<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Store a new user.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $name = $request->input('name');

        //
    }
}

.
3.6. Ejercicios

En los ejercicios de esta parte vamos a continuar con el sitio Web que empezamos para la gestión de un videoclub. Primero añadiremos los controladores y métodos asociados a cada ruta, y posteriormente también completaremos las vistas usando formularios y el sistema de plantillas Blade.
Ejercicio 1 - Controladores (1 punto)

En este primer ejercicio vamos a crear los controladores necesarios para gestionar nuestra aplicación y además actualizaremos el fichero de rutas para que los utilice.

Empezamos por añadir los dos controladores que nos van a hacer falta: CatalogController.php y HomeController.php. Para esto tenéis que utilizar el comando de Artisan que permite crear un controlador vacío (sin métodos).

A continuación vamos a añadir los métodos de estos controladores. En la siguiente tabla resumen podemos ver un listado de los métodos por controlador y las rutas que tendrán asociadas:
Ruta	Controlador	Método
/ 	HomeController 	getHome
catalog 	CatalogController 	getIndex
catalog/show/{id} 	CatalogController 	getShow
catalog/create 	CatalogController 	getCreate
catalog/edit/{id} 	CatalogController 	getEdit

Acordaros que los métodos getShow y getEdit tendrán que recibir como parámetro el $id del elemento a mostrar o editar, por lo que la definición del método en el controlador tendrá que ser como la siguiente:

public function getShow($id)
{
    return view('catalog.show', array('id'=>$id));
}

Por último vamos a cambiar el fichero de rutas routes/web.php para que todas las rutas que teníamos definidas (excepto las de login y logout que las dejaremos como están) apunten a los nuevos métodos de los controladores, por ejemplo:

use App\Http\Controllers\HomeController;
use App\Http\Controllers\CatalogController;

Route::get('/', [HomeController::class, 'getHome']);

El código que teníamos puesto para cada ruta con el return con la generación de la vista lo tenéis que mover al método del controlador correspondiente.
Ejercicio 2 - Completar las vistas (2 puntos)

En este ejercicio vamos a terminar los métodos de los controladores que hemos creado en el ejercicio anterior y además completaremos las vistas asociadas:

Método HomeController@getHome

En este método de momento solo vamos a hacer una redirección a la acción que muestra el listado de películas del catálogo: return redirect()->action([CatalogController::class, 'getIndex']);. Más adelante tendremos que comprobar si el usuario está logueado o no, y en caso de que no lo este redirigirle al formulario de login.

Método CatalogController@getIndex

Este método tiene que mostrar un listado de todas las películas que tiene el videoclub. El listado de películas lo podéis obtener del fichero array_peliculas.php facilitado con los materiales. Este array de películas lo tenéis que copiar como variable miembro de la clase (más adelante las almacenaremos en la base de datos). En el método del controlador simplemente tendremos que modificar la generación de la vista para pasarle este array de películas completo ($this->arrayPeliculas).

Y en la vista correspondiente simplemente tendremos que incluir el siguiente trozo de código en su sección content:

<div class="row">

    @foreach( $arrayPeliculas as $key => $pelicula )
    <div class="col-xs-6 col-sm-4 col-md-3 text-center">

        <a href="{{ url('/catalog/show/' . $key ) }}">
            <img src="{{$pelicula['poster']}}" style="height:200px"/>
            <h4 style="min-height:45px;margin:5px 0 10px 0">
                {{$pelicula['title']}}
            </h4>
        </a>

    </div>
    @endforeach

</div>

Como se puede ver en el código, en primer lugar se crea una fila (usando el sistema de rejilla de Bootstrap) y a continuación se realiza un bucle foreach utilizando la notación de Blade para iterar por todas las películas. Para cada película obtenemos su posición en el array y sus datos asociados, y generamos una columna para mostrarlos. Es importante que nos fijemos en como se itera por los elementos de un array de datos y en la forma de acceder a los valores. Además se ha incluido un enlace para que al pulsar sobre una película nos lleve a la dirección /catalog/show/{$key}, siendo key la posición de esa película en el array.

Método CatalogController@getShow

Este método se utiliza para mostrar la vista detalle de una película. Hemos de tener en cuenta que el método correspondiente recibe un identificador que (de momento) se refiere a la posición de la película en el array. Por lo tanto, tendremos que coger dicha película del array ($this->arrayPeliculas[$id]) y pasársela a la vista.

En esta vista vamos a crear dos columnas, la primera columna para mostrar la imagen de la película y la segunda para incluir todos los detalles. A continuación se incluye la estructura HTML que tendría que tener esta pantalla:

<div class="row">

    <div class="col-sm-4">

        {{-- TODO: Imagen de la película --}}

    </div>
    <div class="col-sm-8">

        {{-- TODO: Datos de la película --}}

    </div>
</div>

En la columna de la izquierda completamos el TODO para insertar la imagen de la película. En la columna de la derecha se tendrán que mostrar todos los datos de la película: título, año, director, resumen y su estado. Para mostrar el estado de la película consultaremos el valor rented del array, el cual podrá tener dos casos:

    En caso de estar disponible (false) aparecerá el estado "Película disponible" y un botón azul para "Alquilar película".
    En caso de estar alquilada (true) aparecerá el estado "Película actualmente alquilada" y un botón rojo para "Devolver película".

Además tenemos que incluir dos botones más, un botón que nos llevará a editar la película y otro para volver al listado de películas.

    Nota: los botones de alquilar/devolver de momento no tienen que funcionar. Acordaros que en Bootstrap podemos transformar un enlace en un botón, simplemente aplicando las clases "btn btn-default" (más info en: http://getbootstrap.com/css/#buttons).

Esta pantalla finalmente tendría el siguiente código:

@section('content')

    <div class="row">

        <div class="col-sm-4">

            <a href="{{ url('/catalog/show/' . $id ) }}">
                <img src="{{$pelicula['poster']}}" style="height:200px"/>
            </a>

        </div>
        <div class="col-sm-8">

            <h4>{{$pelicula['title']}}</h4>
            <h6>A&ntilde;o: {{$pelicula['year']}}</h6>
            <h6>Director: {{$pelicula['director']}}</h6>
            <p><strong>Resumen:</strong> {{$pelicula['synopsis']}}</p>
            <p><strong>Estado: </strong>
                @if($pelicula['rented'])
                    Pel&iacute;cula actualmente alquilada.
                @else
                    Pel&iacute;cula en stock.
                @endif
            </p>

            @if($pelicula['rented'])
                <a class="btn btn-danger" href="#">Devolver pel&iacute;cula</a>
            @else
                <a class="btn btn-primary" href="#">Alquilar pel&iacute;cula</a>
            @endif
            <a class="btn btn-warning" href="{{ url('/catalog/edit/' . $id ) }}">
                <span class="glyphicon glyphicon-pencil" aria-hidden="true"></span>
                Editar pel&iacute;cula</a>
            <a class="btn btn-outline-info" href="{{ action('App\Http\Controllers\CatalogController@getIndex') }}">Volver al listado</a>

        </div>
</div>

@endsection

Método CatalogController@getCreate

Este método devuelve la vista "catalog.create" para añadir una nueva película. Para crear este formulario en la vista correspondiente nos podemos basar en el contenido de la plantilla "catalog_create.php". Esta plantilla tiene una serie de TODOs que hay que completar. En total tendrá que tener los siguientes campos:
Label	Name	Tipo de campo
Título 	title 	text
Año 	year 	text
Director 	director 	text
Poster 	poster 	text
Resumen 	synopsis 	textarea

Además tendrá un botón al final con el texto "Añadir película".

@section('content')

<div class="row" style="margin-top:40px">
   <div class="offset-md-3 col-md-6">
      <div class="card">
         <div class="card-header text-center">
            Añadir película
         </div>
         <div class="card-body" style="padding:30px">

            <form action="{{ url('/catalog/create') }}" method="POST">

	            @csrf

	            <div class="form-group">
	               <label for="title">Título</label>
	               <input type="text" name="title" id="title" class="form-control">
	            </div>

	            <div class="form-group">
	            	<label for="title">A&ntilde;o</label>
	               <input type="number" name="year" id="year">
	            </div>

	            <div class="form-group">
	            	<label for="title">Director</label>
	               <input type="text" name="director" id="director" class="form-control">
	            </div>

	            <div class="form-group">
	            	<label for="title">P&oacute;ster</label>
	               <input type="text" name="poster" id="poster" class="form-control">
	            </div>

	            <div class="form-group">
	               <label for="synopsis">Resumen</label>
	               <textarea name="synopsis" id="synopsis" class="form-control" rows="3"></textarea>
	            </div>

	            <div class="form-group text-center">
	               <button type="submit" class="btn btn-primary" style="padding:8px 100px;margin-top:25px;">
	                   Añadir película
	               </button>
	            </div>

            </form>

         </div>
      </div>
   </div>
</div>

@endsection

    De momento el formulario no funcionará. Más adelante lo terminaremos.

Método CatalogController@getEdit

Este método permitirá modificar el contenido de una película. El formulario será exactamente igual al de añadir película, así que lo podemos copiar y pegar en esta vista y simplemente cambiar los siguientes puntos:

    El título por "Modificar película".
    El texto del botón de envío por "Modificar película".
    Añadir justo debajo de la apertura del formulario el campo oculto para indicar que se va a enviar por PUT. Recordad que Laravel incluye el método {{method_field('PUT')}} que nos ayudará a hacer esto.

De momento no tendremos que hacer nada más, más adelante lo completaremos para que se rellene con los datos de la película a editar.
4. Base de datos

Laravel facilita la configuración y el uso de diferentes tipos de base de datos: MySQL, Postgres, SQLite y SQL Server. En el fichero de configuración (config/database.php) tenemos que indicar todos los parámetros de acceso a nuestras bases de datos y además especificar cual es la conexión que se utilizará por defecto. En Laravel podemos hacer uso de varias bases de datos a la vez, aunque sean de distinto tipo. Por defecto se accederá a la que especifiquemos en la configuración y si queremos acceder a otra conexión lo tendremos que indicar expresamente al realizar la consulta.

En este capítulo veremos como configurar una base de datos, como crear tablas y especificar sus campos desde código, como inicializar la base de datos y como construir consultas tanto de forma directa como a través del ORM llamado Eloquent.
4.1. Configuración inicial

En este primer apartado vamos a ver los primeros pasos que tenemos que dar con Laravel para empezar a trabajar con bases de datos. Para esto vamos a ver a continuación como definir la configuración de acceso, como crear una base de datos y como crear la tabla de migraciones, necesaria para crear el resto de tablas.
Configuración de la Base de Datos

Lo primero que tenemos que hacer para trabajar con bases de datos es completar la configuración. Como ejemplo vamos a configurar el acceso a una base de datos tipo MySQL. Si editamos el fichero con la configuración config/database.php podemos ver en primer lugar la siguiente línea:

'default' => env('DB_CONNECTION', 'mysql'),

Este valor indica el tipo de base de datos a utilizar por defecto. Como vimos en el primer capítulo Laravel utiliza el sistema de variables de entorno para separar las distintas configuraciones de usuario o de máquina. El método env('DB_CONNECTION', 'mysql') lo que hace es obtener el valor de la variable DB_CONNECTION del fichero .env. En caso de que dicha variable no esté definida devolverá el valor por defecto mysql.

En este mismo fichero de configuración, dentro de la sección connections, podemos encontrar todos los campos utilizados para configurar cada tipo de base de datos, en concreto la base de datos tipo mysql tiene los siguientes valores:

'mysql' => [
    'driver'    => 'mysql',
    'host'      => env('DB_HOST', 'localhost'),
    'database'  => env('DB_DATABASE', 'forge'), // Nombre de la base de datos
    'username'  => env('DB_USERNAME', 'forge'), // Usuario de acceso a la bd
    'password'  => env('DB_PASSWORD', ''),      // Contraseña de acceso
    'charset'   => 'utf8',
    'collation' => 'utf8_unicode_ci',
    'prefix'    => '',
    'strict'    => false,
],

Como se puede ver, básicamente los campos que tenemos que configurar para usar nuestra base de datos son: host, database, username y password. El host lo podemos dejar como está si vamos a usar una base de datos local, mientras que los otros tres campos sí que tenemos que actualizarlos con el nombres de la base de datos a utilizar y el usuario y la contraseña de acceso. Para poner estos valores abrimos el fichero .env de la raíz del proyecto y los actualizamos:

DB_CONNECTION=mysql
DB_HOST=localhost
DB_DATABASE=nombre-base-de-datos
DB_USERNAME=nombre-de-usuario
DB_PASSWORD=contraseña-de-acceso

Crear la base de datos

Para crear la base de datos que vamos a utilizar en MySQL podemos utilizar la herramienta PHPMyAdmin que se ha instalado con el paquete XAMPP. Para esto accedemos a la ruta:

http://localhost/phpmyadmin

La cual nos mostrará un panel para la gestión de las bases de datos de MySQL, que nos permite, además de realizar cualquier tipo de consulta SQL, crear nuevas bases de datos o tablas, e insertar, modificar o eliminar los datos directamente. En nuestro caso apretamos en la pestaña "Bases de datos" y creamos una nueva base de datos. El nombre que le pongamos tiene que ser el mismo que el que hayamos indicado en el fichero de configuración de Laravel.
Tabla de migraciones

A continuación vamos a crear la tabla de migraciones. En la siguiente sección veremos en detalle que es esto, de momento solo decir que Laravel utiliza las migraciones para poder definir y crear las tablas de la base de datos desde código, y de esta manera tener un control de las versiones de las mismas.

Para poder empezar a trabajar con las migraciones es necesario en primer lugar crear la tabla de migraciones. Para esto tenemos que ejecutar el siguiente comando de Artisan:

php artisan migrate:install

    Si nos diese algún error tendremos que revisar la configuración que hemos puesto de la base de datos y si hemos creado la base de datos con el nombre, usuario y contraseña indicado.

Si todo funciona correctamente ahora podemos ir al navegador y acceder de nuevo a nuestra base de datos con PHPMyAdmin, podremos ver que se nos habrá creado la tabla migrations. Con esto ya tenemos configurada la base de datos y el acceso a la misma. En las siguientes secciones veremos como añadir tablas y posteriormente como realizar consultas.
4.2. Migraciones

Las migraciones son un sistema de control de versiones para bases de datos. Permiten que un equipo trabaje sobre una base de datos añadiendo y modificando campos, manteniendo un histórico de los cambios realizados y del estado actual de la base de datos. Las migraciones se utilizan de forma conjunta con la herramienta Schema builder (que veremos en la siguiente sección) para gestionar el esquema de base de datos de la aplicación.

La forma de funcionar de las migraciones es crear ficheros (PHP) con la descripción de la tabla a crear y posteriormente, si se quiere modificar dicha tabla se añadiría una nueva migración (un nuevo fichero PHP) con los campos a modificar. Artisan incluye comandos para crear migraciones, para ejecutar las migraciones o para hacer rollback de las mismas (volver atrás).
Crear una nueva migración

Para crear una nueva migración se utiliza el comando de Artisan make:migration, al cual le pasaremos el nombre del fichero a crear y el nombre de la tabla:

php artisan make:migration create_users_table --create=users

Esto nos creará un fichero de migración en la carpeta database/migrations con el nombre <TIMESTAMP>_create_users_table.php. Al añadir un timestamp a las migraciones el sistema sabe el orden en el que tiene que ejecutar (o deshacer) las mismas.

Si lo que queremos es añadir una migración que modifique los campos de una tabla existente tendremos que ejecutar el siguiente comando:

php artisan make:migration add_votes_to_user_table --table=users

En este caso se creará también un fichero en la misma carpeta, con el nombre <TIMESTAMP>_add_votes_to_user_table.php pero preparado para modificar los campos de dicha tabla.

Por defecto, al indicar el nombre del fichero de migraciones se suele seguir siempre el mismo patrón (aunque el realidad el nombre es libre). Si es una migración que crea una tabla el nombre tendrá que ser create_<table-name>_table y si es una migración que modifica una tabla será <action>_to_<table-name>_table.
Estructura de una migración

El fichero o clase PHP generada para una migración siempre tiene una estructura similar a la siguiente:

<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateUsersTable extends Migration 
{
    /**
     * Run the migrations.
     * @return void
     */
    public function up()
    {
        //
    }

    /**
     * Reverse the migrations.
     * @return void
     */
    public function down()
    {
        //
    }
}

En el método up es donde tendremos crear o modificar la tabla, y en el método down tendremos que deshacer los cambios que se hagan en el up (eliminar la tabla o eliminar el campo que se haya añadido). Esto nos permitirá poder ir añadiendo y eliminando cambios sobre la base de datos y tener un control o histórico de los mismos.
Ejecutar migraciones

Después de crear una migración y de definir los campos de la tabla (en la siguiente sección veremos como especificar esto) tenemos que lanzar la migración con el siguiente comando:

php artisan migrate

    Si nos aparece el error "class not found" lo podremos solucionar llamando a composer dump-autoload y volviendo a lanzar las migraciones.

Este comando aplicará la migración sobre la base de datos. Si hubiera más de una migración pendiente se ejecutarán todas. Para cada migración se llamará a su método up para que cree o modifique la base de datos. Posteriormente en caso de que queramos deshacer los últimos cambios podremos ejecutar:

php artisan migrate:rollback

# O si queremos deshacer todas las migraciones
php artisan migrate:reset

Un comando interesante cuando estamos desarrollando un nuevo sitio web es migrate:refresh, el cual deshará todos los cambios y volver a aplicar las migraciones:

php artisan migrate:refresh

Además si queremos comprobar el estado de las migraciones, para ver las que ya están instaladas y las que quedan pendientes, podemos ejecutar:

php artisan migrate:status

4.3. Schema Builder

Una vez creada una migración tenemos que completar sus métodos up y down para indicar la tabla que queremos crear o el campo que queremos modificar. En el método down siempre tendremos que añadir la operación inversa, eliminar la tabla que se ha creado en el método up o eliminar la columna que se ha añadido. Esto nos permitirá deshacer migraciones dejando la base de datos en el mismo estado en el que se encontraban antes de que se añadieran.

Para especificar la tabla a crear o modificar, así como las columnas y tipos de datos de las mismas, se utiliza la clase Schema. Esta clase tiene una serie de métodos que nos permitirá especificar la estructura de las tablas independientemente del sistema de base de datos que utilicemos.
Crear y borrar una tabla

Para añadir una nueva tabla a la base de datos se utiliza el siguiente constructor:

Schema::create('users', function (Blueprint $table) {
    $table->increments('id');
});

Donde el primer argumento es el nombre de la tabla y el segundo es una función que recibe como parámetro un objeto del tipo Blueprint que utilizaremos para configurar las columnas de la tabla.

En la sección down de la migración tendremos que eliminar la tabla que hemos creado, para esto usaremos alguno de los siguientes métodos:

Schema::drop('users');

Schema::dropIfExists('users');

Al crear una migración con el comando de Artisan make:migration ya nos viene este código añadido por defecto, la creación y eliminación de la tabla que se ha indicado y además se añaden un par de columnas por defecto (id y timestamps).
Añadir columnas

El constructor Schema::create recibe como segundo parámetro una función que nos permite especificar las columnas que va a tener dicha tabla. En esta función podemos ir añadiendo todos los campos que queramos, indicando para cada uno de ellos su tipo y nombre, y además si queremos también podremos indicar una serie de modificadores como valor por defecto, índices, etc. Por ejemplo:

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

Schema define muchos tipos de datos que podemos utilizar para definir las columnas de una tabla, algunos de los principales son:
Comando	Tipo de campo
$table->boolean('confirmed'); 	BOOLEAN
$table->enum('choices', array('foo', 'bar')); 	ENUM
$table->float('amount'); 	FLOAT
$table->increments('id'); 	Clave principal tipo INTEGER con Auto-Increment
$table->integer('votes'); 	INTEGER
$table->mediumInteger('numbers'); 	MEDIUMINT
$table->smallInteger('votes'); 	SMALLINT
$table->tinyInteger('numbers'); 	TINYINT
$table->string('email'); 	VARCHAR
$table->string('name', 100); 	VARCHAR con la longitud indicada
$table->text('description'); 	TEXT
$table->timestamp('added_on'); 	TIMESTAMP
$table->timestamps(); 	Añade los timestamps "created_at" y "updated_at"
->nullable() 	Indicar que la columna permite valores NULL
->default($value) 	Declare a default value for a column
->unsigned() 	Añade UNSIGNED a las columnas tipo INTEGER

Los tres últimos se pueden combinar con el resto de tipos para crear, por ejemplo, una columna que permita nulos, con un valor por defecto y de tipo unsigned.

Para consultar todos los tipos de datos que podemos utilizar podéis consultar la documentación de Laravel en:

http://laravel.com/docs/migrations#columns
Añadir índices

Schema soporta los siguientes tipos de índices:
Comando	Descripción
$table->primary('id'); 	Añadir una clave primaria
$table->primary(array('first', 'last')); 	Definir una clave primaria compuesta
$table->unique('email'); 	Definir el campo como UNIQUE
$table->index('state'); 	Añadir un índice a una columna

En la tabla se especifica como añadir estos índices después de crear el campo, pero también permite indicar estos índices a la vez que se crea el campo:

$table->string('email')->unique();

Claves ajenas

Con Schema también podemos definir claves ajenas entre tablas:

$table->bigInteger('user_id')->unsigned();
$table->foreign('user_id')->references('id')->on('users');

En este ejemplo en primer lugar añadimos la columna "user_id" de tipo UNSIGNED INTEGER (siempre tendremos que crear primero la columna sobre la que se va a aplicar la clave ajena). A continuación creamos la clave ajena entre la columna "user_id" y la columna "id" de la tabla "users".

    La columna con la clave ajena tiene que ser del mismo tipo que la columna a la que apunta. Si por ejemplo creamos una columna a un índice auto-incremental tendremos que especificar que la columna sea unsigned para que no se produzcan errores.

También podemos especificar las acciones que se tienen que realizar para "on delete" y "on update":

$table->foreign('user_id')
      ->references('id')->on('users')
      ->onDelete('cascade');

Para eliminar una clave ajena, en el método down de la migración tenemos que utilizar el siguiente código:

$table->dropForeign('posts_user_id_foreign');

Para indicar la clave ajena a eliminar tenemos que seguir el siguiente patrón para especificar el nombre <tabla>_<columna>_foreign. Donde "tabla" es el nombre de la tabla actual y "columna" el nombre de la columna sobre la que se creo la clave ajena.
4.4. Modelos de datos mediante ORM

El mapeado objeto-relacional (más conocido por su nombre en inglés, Object-Relational mapping, o por sus siglas ORM) es una técnica de programación para convertir datos entre un lenguaje de programación orientado a objetos y una base de datos relacional como motor de persistencia. Esto posibilita el uso de las características propias de la orientación a objetos, podremos acceder directamente a los campos de un objeto para leer los datos de una base de datos o para insertarlos o modificarlos.

Laravel incluye su propio sistema de ORM llamado Eloquent, el cual nos proporciona una manera elegante y fácil de interactuar con la base de datos. Para cada tabla de la base datos tendremos que definir su correspondiente modelo, el cual se utilizará para interactuar desde código con la tabla.
Definición de un modelo

Por defecto los modelos se guardarán como clases PHP dentro de la carpeta app/Models, sin embargo Laravel nos da libertad para colocarlos en otra carpeta si queremos, como por ejemplo la carpeta app/Models. Pero en este caso tendremos que asegurarnos de indicar correctamente el espacio de nombres.

Para definir un modelo que use Eloquent únicamente tenemos que crear una clase que herede de la clase Model:

<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    //...
}

Sin embargo es mucho más fácil y rápido crear los modelos usando el comando make:model de Artisan:

php artisan make:model User

Este comando creará el fichero User.php dentro de la carpeta app con el código básico de un modelo que hemos visto en el ejemplo anterior.
Convenios en Eloquent
Nombre

En general el nombre de los modelos se pone en singular con la primera letra en mayúscula, mientras que el nombre de las tablas suele estar en plural. Gracias a esto, al definir un modelo no es necesario indicar el nombre de la tabla asociada, sino que Eloquent automáticamente buscará la tabla transformando el nombre del modelo a minúsculas y buscando su plural (en inglés). En el ejemplo anterior que hemos creado el modelo User buscará la tabla de la base de datos llamada users y en caso de no encontrarla daría un error.

Si la tabla tuviese otro nombre lo podemos indicar usando la propiedad protegida $table del modelo:

<?php 
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $table = 'my_users';
}

Clave primaria

Laravel también asume que cada tabla tiene declarada una clave primaria con el nombre id. En el caso de que no sea así y queramos cambiarlo tendremos que sobrescribir el valor de la propiedad protegida $primaryKey del modelo, por ejemplo: protected $primaryKey = 'my_id';.

    Es importante definir correctamente este valor ya que se utiliza en determinados métodos de Eloquent, como por ejemplo para buscar registros o para crear las relaciones entre modelos.

Timestamps

Otra propiedad que en ocasiones tendremos que establecer son los timestamps automáticos. Por defecto Eloquent asume que todas las tablas contienen los campos updated_at y created_at (los cuales los podemos añadir muy fácilmente con Schema añadiendo $table->timestamps() en la migración). Estos campos se actualizarán automáticamente cuando se cree un nuevo registro o se modifique. En el caso de que no queramos utilizarlos (y que no estén añadidos a la tabla) tendremos que indicarlo en el modelo o de otra forma nos daría un error. Para indicar que no los actualice automáticamente tendremos que modificar el valor de la propiedad pública $timestamps a false, por ejemplo: public $timestamps = false;.

A continuación se muestra un ejemplo de un modelo de Eloquent en el que se añaden todas las especificaciones que hemos visto:

<?php 
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $table = 'my_users';
    protected $primaryKey = 'my_id'
    public $timestamps = false;
}

Uso de un modelo de datos

Una vez creado el modelo ya podemos empezar a utilizarlo para recuperar datos de la base de datos, para insertar nuevos datos o para actualizarlos. El sitio correcto donde realizar estas acciones es en el controlador, el cual se los tendrá que pasar a la vista ya preparados para su visualización.

Es importante que para su utilización indiquemos al inicio de la clase el espacio de nombres del modelo o modelos a utilizar. Por ejemplo, si vamos a usar los modelos User y Orders tendríamos que añadir:

use App\Models\User;
use App\Models\Orders;

Consultar datos

Para obtener todas las filas de la tabla asociada a un modelo usaremos el método all():

$users = User::all();

foreach( $users as $user ) {
    echo $user->name;
}

Este método nos devolverá un array de resultados, donde cada item del array será una instancia del modelo User. Gracias a esto al obtener un elemento del array podemos acceder a los campos o columnas de la tabla como si fueran propiedades del objeto ($user->name).

    Nota: Todos los métodos que se describen en la sección de "Constructor de consultas" y en la documentación de Laravel sobre "Query Builder" también se pueden utilizar en los modelos Eloquent. Por lo tanto podremos utilizar where, orWhere, first, get, orderBy, groupBy, having, skip, take, etc. para elaborar las consultas.

Eloquent también incorpora el método find($id) para buscar un elemento a partir del identificador único del modelo, por ejemplo:

$user = User::find(1);
echo $user->name;

Si queremos que se lance una excepción cuando no se encuentre un modelo podemos utilizar los métodos findOrFail o firstOrFail. Esto nos permite capturar las excepciones y mostrar un error 404 cuando sucedan.

$model = User::findOrFail(1);

$model = User::where('votes', '>', 100)->firstOrFail();

A continuación se incluyen otros ejemplos de consultas usando Eloquent con algunos de los métodos que ya habíamos visto en la sección "Constructor de consultas":

// Obtener 10 usuarios con más de 100 votos
$users = User::where('votes', '>', 100)->take(10)->get();

// Obtener el primer usuario con más de 100 votos
$user = User::where('votes', '>', 100)->first();

También podemos utilizar los métodos agregados para calcular el total de registros obtenidos, o el máximo, mínimo, media o suma de una determinada columna. Por ejemplo:

$count = User::where('votes', '>', 100)->count();
$price = Orders::max('price');
$price = Orders::min('price');
$price = Orders::avg('price');
$total = User::sum('votes');

Insertar datos

Para añadir una entrada en la tabla de la base de datos asociada con un modelo simplemente tenemos que crear una nueva instancia de dicho modelo, asignar los valores que queramos y por último guardarlos con el método save():

$user = new User;
$user->name = 'Juan';
$user->save();

Para obtener el identificador asignado en la base de datos después de guardar (cuando se trate de tablas con índice auto-incremental), lo podremos recuperar simplemente accediendo al campo id del objeto que habíamos creado, por ejemplo:

$insertedId = $user->id;

Actualizar datos

Para actualizar una instancia de un modelo es muy sencillo, solo tendremos que recuperar en primer lugar la instancia que queremos actualizar, a continuación modificarla y por último guardar los datos:

$user = User::find(1);
$user->email = 'juan@gmail.com';
$user->save();

Borrar datos

Para borrar una instancia de un modelo en la base de datos simplemente tenemos que usar su método delete():

$user = User::find(1);
$user->delete();

Si por ejemplo queremos borrar un conjunto de resultados también podemos usar el método delete() de la forma:

$affectedRows = User::where('votes', '>', 100)->delete();

Más información

Para más información sobre como crear relaciones entre modelos, eager loading, etc. podéis consultar directamente la documentación de Laravel en:

http://laravel.com/docs/eloquent
4.5. Inicialización de la base de datos (Database Seeding)

Laravel también facilita la inserción de datos iniciales o datos semilla en la base de datos. Esta opción es muy útil para tener datos de prueba cuando estamos desarrollando una web o para crear tablas que ya tienen que contener una serie de datos en producción.

Los ficheros de "semillas" se encuentran en la carpeta database/seeders. Por defecto Laravel incluye el fichero DatabaseSeeder con el siguiente contenido:

<?php

use Illuminate\Database\Seeder;
use Illuminate\Database\Eloquent\Model;

class DatabaseSeeder extends Seeder
{
    /**
     * Run the database seeds.
     * @return void
     */
    public function run()
    {
        //...
    }
}

Al lanzar la incialización se llamará por defecto al método run de la clase DatabaseSeeder. Desde aquí podemos crear las semillas de varias formas:

    Escribir el código para insertar los datos dentro del propio método run.
    Crear otros métodos dentro de la clase DatabaseSeeder y llamarlos desde el método run. De esta forma podemos separar mejor las inicializaciones.
    Crear otros ficheros Seeder y llamarlos desde el método run es la clase principal.

Según lo que vayamos a hacer nos puede interesar una opción u otra. Por ejemplo, si el código que vamos a escribir es poco nos puede sobrar con las opciones 1 o 2, sin embargo si vamos a trabajar bastante con las inicializaciones quizás lo mejor es la opción 3.

A continuación se incluye un ejemplo de la opción 1:

<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

class DatabaseSeeder extends Seeder
{
    /**
     * Run the database seeders.
     *
     * @return void
     */
    public function run()
    {
        DB::table('users')->insert([
            'name' => Str::random(10),
            'email' => Str::random(10).'@gmail.com',
            'password' => Hash::make('password'),
        ]);
    }
}

Como se puede ver en el ejemplo en general tendremos que eliminar primero los datos de la tabla en cuestión y posteriormente añadir los datos. Para insertar datos en una tabla podemos utilizar el "Constructor de consultas" y "Eloquent ORM".
Crear ficheros semilla

Como hemos visto en el apartado anterior, podemos crear más ficheros o clases semilla para modularizar mejor el código de las inicializaciones. De esta forma podemos crear un fichero de semillas para cada una de las tablas o modelos de datos que tengamos.

En la carpeta database/seeders podemos añadir más ficheros PHP con clases que extiendan de Seeder para definir nuestros propios ficheros de "semillas". El nombre de los ficheros suele seguir el mismo patrón <nombre-tabla>TableSeeder, por ejemplo "UsersTableSeeder". Artisan incluye un comando que nos facilitará crear los ficheros de semillas y que además incluirán las estructura base de la clase. Por ejemplo, para crear el fichero de inicialización de la tabla de usuarios haríamos:

php artisan make:seeder UsersTableSeeder

Para que esta nueva clase se ejecute tenemos que llamarla desde el método run de la clase principal DatabaseSeeder de la forma:

class DatabaseSeeder extends Seeder 
{
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Support\Facades\Schema;

    public function run()
    {
        Model::unguard();
        Schema::disableForeignKeyConstraints();

        $this->call(UsersTableSeeder::class);

        Model::reguard();

        Schema::enableForeignKeyConstraints();
 }
}

El método call lo que hace es llamar al método run de la clase indicada. Además en el ejemplo hemos añadido las llamadas a unguard y a reguard, que lo que hacen es desactivar y volver a activar (respectivamente) la inserción de datos masiva o por lotes.
Ejecutar la inicialización de datos

Una vez definidos los ficheros de semillas, cuando queramos ejecutarlos para rellenar de datos la base de datos tendremos que usar el siguiente comando de Artisan:

php artisan db:seed

4.6. Constructor de consultas (Query Builder)

Laravel incluye una serie de clases que nos facilita la construcción de consultas y otro tipo de operaciones con la base de datos. Además, al utilizar estas clases, creamos una notación mucho más legible, compatible con todos los tipos de bases de datos soportados por Laravel y que nos previene de cometer errores o de ataques por inyección de código SQL.
Consultas

Para realizar una "Select" que devuelva todas las filas de una tabla utilizaremos el siguiente código:

$users = DB::table('users')->get();

foreach ($users as $user)
{
    echo $user->name;
}

En el ejemplo se utiliza el constructor DB::tabla indicando el nombre de la tabla sobre la que se va a realizar la consulta, y por último se llama al método get() para obtener todas las filas de la misma.

Si queremos obtener un solo elemento podemos utilizar first en lugar de get, de la forma:

$user = DB::table('users')->first();

echo $user->name;

Clausula where

Para filtrar los datos usamos la clausula where, indicando el nombre de la columna y el valor a filtrar:

$user = DB::table('users')->where('name', 'Pedro')->get();

echo $user->name;

En este ejemplo, la clausula where filtrará todas las filas cuya columna name sea igual a Pedro. Si queremos realizar otro tipo de filtrados, como columnas que tengan un valor mayor (>), mayor o igual (>=), menor (<), menor o igual (<=), distinto del indicado (<>) o usar el operador like, lo podemos indicar como segundo parámetro de la forma:

$users = DB::table('users')->where('votes', '>', 100)->get();

$users = DB::table('users')->where('status', '<>', 'active')->get();

$users = DB::table('users')->where('name', 'like', 'T%')->get();

Si añadimos más clausulas where a la consulta por defecto se unirán mediante el operador lógico AND. En caso de que queramos utilizar el operador lógico OR lo tendremos que realizar usando orWhere de la forma:

$users = DB::table('users')
                    ->where('votes', '>', 100)
                    ->orWhere('name', 'Pedro')
                    ->get();

orderBy / groupBy / having_

También podemos utilizar los métodos orderBy, groupBy y having en las consultas:

$users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->groupBy('count')
                    ->having('count', '>', 100)
                    ->get();

Offset / Limit

Si queremos indicar un offset o limit lo realizaremos mediante los métodos skip (para el offset) y take (para limit), por ejemplo:

$users = DB::table('users')->skip(10)->take(5)->get();

Transacciones

Laravel también permite crear transacciones sobre un conjunto de operaciones:

DB::transaction(function()
{
    DB::table('users')->update(array('votes' => 1));

    DB::table('posts')->delete();
});

En caso de que se produzca cualquier excepción en las operaciones que se realizan en la transacción se desharían todos los cambios aplicados hasta ese momento de forma automática.
Más informacion

Para más información sobre la construcción de Querys (join, insert, update, delete, agregados, etc.) podéis consultar la documentación de Laravel en su sitio web:

http://laravel.com/docs/queries
4.7. Ejercicios

En estos ejercicios vamos a continuar con el proyecto del videoclub que habíamos empezado en sesiones anteriores y le añadiremos todo lo referente a la gestión de la base de datos.

Ejercicio 1 - Configuración de la base de datos y migraciones (1 punto)

En primer lugar vamos a configurar correctamente la base de datos. Para esto tenemos que actualizar el fichero .env para indicar que vamos a usar una base de datos tipo MySQL llamada "videoclub" junto con el nombre de usuario y contraseña de acceso.

A continuación abrimos PHPMyAdmin y creamos la nueva base de datos llamada videoclub. Para comprobar que todo se ha configurado correctamente vamos a un terminal en la carpeta de nuestro proyecto y ejecutamos el comando que crea la tabla de migraciones. Si todo va bien podremos actualizar desde PHPMyAdmin y comprobar que se ha creado esta tabla dentro de nuestra nueva base de datos.

    Si nos diese algún error tendremos que revisar los valores indicados en el fichero .env. En caso de ser correctos es posible que también tengamos que reiniciar el servidor o terminal que tengamos abierto.

Ahora vamos a crear la tabla que utilizaremos para almacenar el catálogo de películas. Ejecuta el comando de Artisan para crear la migración llamada create_movies_table para la tabla movies. Una vez creado edita este fichero para añadir todos los campos necesarios, estos son:

Campo	Tipo	Valor por defecto
id 	Autoincremental 	
title 	String 	
year 	Year 	
director 	String de longitud 64 	
poster 	String 	
rented 	Booleano 	false
synopsis 	Text 	
timestamps 	Timestamps de Eloquent 	 

    Recuerda que en el método down de la migración tienes que deshacer los cambios que has hecho en el método up, en este caso sería eliminar la tabla.

Por último ejecutaremos el comando de Artisan que añade las nuevas migraciones y comprobaremos que la tabla se ha creado correctamente con los campos que le hemos indicado.

Ejercicio 2 - Modelo de datos (0.5 puntos)

En este ejercicio vamos a crear el modelo de datos asociado con la tabla movies. Para esto usaremos el comando apropiado de Artisan para crear el modelo llamado Movie.

Una vez creado este fichero lo abriremos y comprobaremos que el nombre de la clase sea el correcto y que herede de la clase Model. Y ya está, no es necesario hacer nada más, el cuerpo de la clase puede estar vacío ({}), todo lo demás se hace automáticamente!

Ejercicio 3 - Semillas (1 punto)

Ahora vamos a proceder a rellenar la tabla de la base de datos con los datos iniciales. Para esto editamos el fichero de semillas situado en database/seeders/DatabaseSeeder.php y seguiremos los siguientes pasos:

    Creamos un método privado de clase llamado seedCatalog() que se tendrá que llamar desde el método run de la forma:

    public function run()
    {
      self::seedCatalog();
      $this->command->info('Tabla catálogo inicializada con datos!');
    }

    Movemos el array de películas que se facilitaba en los materiales y que habíamos copiado dentro del controlador CatalogController a la clase de semillas (DatabaseSeeder.php), guardándolo como variable privada de la clase.

    Dentro del nuevo método seedCatalog() realizamos las siguientes acciones:
        En primer lugar borramos el contenido de la tabla movies con Movie::truncate();.
        Y a continuación añadimos el siguiente código:

        foreach( self::$arrayPeliculas as $pelicula ) {
            $p = new Movie;
            $p->title = $pelicula['title'];
            $p->year = $pelicula['year'];
            $p->director = $pelicula['director'];
            $p->poster = $pelicula['poster'];
            $p->rented = $pelicula['rented'];
            $p->synopsis = $pelicula['synopsis'];
            $p->save();
        }

Por último tendremos que ejecutar el comando de Artisan que procesa las semillas y una vez realizado comprobaremos que se rellenado la tabla movies con el listado de películas.

    Si te aparece el error "Fatal error: Class 'Movie' not found" revisa si has indicado el espacio de nombres del modelo que vas a utilizar (use App\Models\Movie;).

Ejercicio 4 - Uso de la base de datos (1 punto)

En este último ejercicio vamos a actualizar los métodos del controlador CatalogController para que obtengan los datos desde la base de datos. Seguiremos los siguientes pasos:

    Modificar el método getIndex para que obtenga toda la lista de películas desde la base de datos usando el modelo Movie y que se pase a la vista ese listado.

    Modificar el método getShow para que obtenga la película pasada por parámetro usando el método findOrFail y se pase a la vista dicha película.

    Modificar el método getEdit para que obtenga la película pasada por parámetro usando el método findOrFail y se pase a la vista dicha película.

    Si al probarlo te aparece el error "Class 'App\Http\Controllers\Movie' not found" revisa si has indicado el espacio de nombres del modelo que vas a utilizar (use App\Models\Movie;).

Ya no necesitaremos más el array de películas ($arrayPeliculas) que habíamos puesto en el controlador, así que lo podemos comentar o eliminar.

Ahora tendremos que actualizar las vistas para que en lugar de acceder a los datos del array los obtenga del objeto con la película. Para esto cambiaremos en todos los sitios donde hayamos puesto $pelicula['campo'] por $pelicula->campo.

Además, en la vista catalog/index.blade.php, en vez de utilizar el índice del array ($key) como identificador para crear el enlace a catalog/show/{id}, tendremos que utilizar el campo id de la película ($pelicula->id). Lo mismo en la vista catalog/show.blade.php, para generar el enlace de editar película tendremos que añadir el identificador de la película a la ruta catalog/edit.
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

