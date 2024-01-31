# Policies

## Generando Policies

Las `policies` son clases que organizan la lógica de **autorización** referente a un determinado modelo o recurso.

En nuestro caso, organizaremos la autorización sobre el modelo `App\Models\Curriculo` con el correspondiente `policy` `App\Policies\CurriculoPolicy` para autorizar las acciones del usuario, tales como crear o actualizar curriculos.

Podemos generar la `policy` usando el comando _Artisan_ `make:policy` que podemos asociar al modelo `Curriculo` con el modificador `--model`:

```bash
php artisan make:policy CurriculoPolicy --model=Curriculo
```

## Registando Policies

Una vez que la clase `policy` ha sido creada, necesita ser registrada en `App\Providers\AuthServiceProvider` en el _array_ `$policies`:

```php
<?php

namespace App\Providers;

use App\Models\Curriculo;
use App\Policies\CurriculoPolicy;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Gate;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        Curriculo::class => CurriculoPolicy::class,
    ];

    /**
     * Register any application authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        //
    }
}
```

> Fíjate que hemos eliminado las autorizaciones con Gates definidas en el apartado anterior.

## Creando Policies

### Policy en Métodos

Una vez que la clase `policy` se ha registrado, se pueden crear métodos para autorizar acciones. Por ejemplo, vamos a definir el método `update` en nuestro `CurriculoPolicy` que determinará si un `User` podrá modificar una instancia de `Curriculo`.

Este método `update` recibirá una instancia de `User` y una instancia `Curriculo` como argumentos, y devolverá `true` o `false` indicando si el usuario está o no autorizado para actualizar el `Curriculo`. En nuestro caso, verificaremos que el `id` del usuario autenticado es el mismo que el valor del atributo `user_id` del curriculo:

```php
<?php

namespace App\Policies;

use App\Models\Curriculo;
use App\Models\User;
use Illuminate\Auth\Access\HandlesAuthorization;

class CurriculoPolicy
{
    /**
     * Determine whether the user can update the model.
     *
     * @param  \App\Models\User  $user
     * @param  \App\Models\Curriculo  $curriculo
     * @return \Illuminate\Auth\Access\Response|bool
     */
    public function update(User $user, Curriculo $curriculo)
    {
        return $user->id === $curriculo->user_id;
    }
}
```

Como hemos usado la opción `--model` al generar la `policy` con el comando _Artisan_, la clase ya contendrá métodos para las acciones `viewAny`, `view`, `create`, `update`, `delete`, `restore`, y `forceDelete`.

### Métodos sin modelos

Hay ocasiones, como ocurre con la acción `create` en el que no se recibe ninguna instancia del modelo y únicamente recibimos la instancia del usuario autenticado. Vamos a suponer que el usuario con el `id = 1` es el **administrador** y que únicamente le vamos a permitir a ese administrador la creación de registros curriculos:

```php
    /**
     * Determine whether the user can create models.
     *
     * @param  \App\Models\User  $user
     * @return \Illuminate\Auth\Access\Response|bool
     */
    public function create(User $user)
    {
        return $user->email === env(ADMIN_EMAIL);
    }
```

### Filtros en las Policies

Algunos usuarios deberán estar autorizados a realizar cualquiera de las acciones asociadas a una Policy. Para ello, definiremos un método `before` que se ejecutará antes que cualquier otro método de la `policy`. Esta característica se suele utilizar para autorizar a los administradores a realizar cualquier acción:

```php
use App\Models\User;

/**
 * Perform pre-authorization checks.
 *
 * @param  \App\Models\User  $user
 * @param  string  $ability
 * @return void|bool
 */
public function before(User $user, $ability)
{
    if($user->email === env(ADMIN_EMAIL)) return true;
}
```

> El método `before()` de una clase `policy` no será llamado si la clase no contiene un método que coincida con el nombre de la capacidad que debe ser verificada.

## Autorizando acciones

### A través del modelo User

El modelo `User` incluye 2 métodos que nos ayudarán a autorizar las acciones: `can()` y `cannot()`. Estos métodos reciben el nombre de la acción que queremos autorizar y la instancia del modelo. Por ejemplo, permitirá determinar si un usuario está autorizado a actualizar (`update`) una determinada instancia del modelo `Curriculo`:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\Curriculo;
use Illuminate\Http\Request;

class CurriculoController extends Controller
{
    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Models\Curriculo  $curriculo
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, Curriculo $curriculo)
    {
        abort_if ($request->user()->cannot('update', $curriculo), 403);

        // Actualiza el curriculo...
    }
}
```

Si existe una `policy` registrada para ese modelo, el método `can()` llamará automáticamente a la `policy` apropiada y devolverá un valor _booleano_ como resultado. Si, por el contrario, no se ha definido esa policy para el modelo, el método `can()` intentará llamar al Gate correspondiente al nombre de la acción.

#### Acciones que no requieren modelos

Recordemos que algunas acciones pueden corresponder a métodos que no requieren de una instancia previa del modelo, por ejemplo el método `create()`. En estas situaciones, podemos enviar, como parámetro, el nombre de la clase al método `can()`.

```php
$request->user()->cannot('create', Curriculo::class)
```

de la siguiente forma:

```php
    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        abort_if ($request->user()->cannot('create', Curriculo::class), 403);

        // Crea el curriculo...
    }
```
### A través de Controller Helpers

Además de los método provistos por el modelo `User`, _Laravel_ ofrece un método `authorize` para cualquiera de los controladores que extiendan la clase base `App\Http\Controllers\Controller`.

El método `authorize` lanzará una excepción `Illuminate\Auth\Access\AuthorizationException` en el caso de que el usuario no tenga permisos para realizar la acción, que será convertida automáticamente a una respuesta _HTTP_ con un código de estado `403`:

```php
<?php

namespace App\Http\Controllers\API;

use App\Http\Controllers\Controller;
use App\Models\Curriculo;
use App\Http\Resources\CurriculoResource;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

class CurriculoController extends Controller
{
    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Models\Curriculo  $curriculo
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, Curriculo $curriculo)
    {
        /* 
        abort_if ($request->user()->cannot('update', $curriculo), 403);
        */

        $this->authorize('update', $curriculo);
        $curriculoData = json_decode($request->getContent(), true);
        $curriculo->update($curriculoData);

        return new CurriculoResource($curriculo);
    }
}
```

#### Acciones que no requieren una instancia del modelo

Como se mencionó previamente, algunos métodos, como `create` no requieren una instancia del modelo. En ese caso, deberemos pasar el nombre de la clase al método `authorize`. El nombre de la clase será necesario para determinar la clase `policy` a utilizar para la autorización:

```php
    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        /* 
        abort_if ($request->user()->cannot('update', $curriculo), 403);
        */

        $this->authorize('create', Curriculo::class);
        
        $curriculo = json_decode($request->getContent(), true);

        $curriculo = Curriculo::create($curriculo);

        return new CurriculoResource($curriculo);
    }
```

### Autorizando controladores de recursos

Como estamos utilizando **controladores de recursos**, podemos hacer uso del método `authorizeResource` en el constructor del controlador para asociar las acciones a las policies.

El método `authorizeResource` acepta el nombre de la clase  como primer argumento y, como segundo argumento, el nombre del parámetro de la ruta que contendrá el identificador del modelo. Para que esto funcione, debemos asegurarnos de que nuestro controlador de recursos se creó con la opción `--model`:

```php
<?php

namespace App\Http\Controllers\API;

use App\Http\Controllers\Controller;
use App\Models\Curriculo;
use App\Http\Resources\CurriculoResource;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

class CurriculoController extends Controller
{
    /**
     * Create the controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->authorizeResource(Curriculo::class, 'curriculo');
    }
```

Los métodos del controlador serán mapeados a los correspondientes métodos de la Policy:

Método del controlador |	Método de la Policy
--|--
index | viewAny
show | view
create | create
store | create
edit | update
update | update
destroy | delete

Cuando las peticiones son enrutadas hacia el método del controlador, el método de la `Policy` relacionado será invocado automáticamente antes de que el método del controlador se ejecute.
