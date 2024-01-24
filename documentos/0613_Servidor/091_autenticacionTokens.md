# Autenticación por tokens

## Backend

## Instalación

**No es necesario instalar nada más**, ya que todo lo necesario lo instalamos al instalar _Laravel Breeze_, cueando ejecutamos `php artisan breeze:install api`

## Generando tokens de API

Para una _SPA_ (Single Page Application), como la que estamos utilizando con _React-Admin_, _Sanctum_ utiliza **cookies** y **sesiones**. No obstante, _Sanctum_ también puede emitir **tokens** que pueden usarse para autenticar solicitudes de _API_ a su aplicación.

Al realizar solicitudes utilizando _tokens_ de _API_, el _token_ debe incluirse en el `Header Authorization` como un _token_ `Bearer`.

Para comenzar a emitir tokens para los usuarios, debemos asegurarnos que el modelo `User` utiliza `Laravel\Sanctum\HasApiTokens`:

```php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

Para emitir un _token_, puede utilizar el método `createToken()`, que devuelve una instancia de `Laravel\Sanctum\NewAccessToken`. Los `tokens` de API se codifican mediante _hash_ `SHA-256` antes de almacenarse en su base de datos, pero puede acceder al valor de texto sin formato del _token_ utilizando la propiedad `plainTextToken` de la instancia `NewAccessToken`. Debe mostrar este valor al usuario inmediatamente después de que se haya creado el token.

### Controlador de tokens
En primer lugar, crearemos el controlador `TokenController`, que incluirá la lógica de emisión y borrado de cada _personal-access-token_, con el siguiente comando _artisan_:

```bash
php artisan make:controller API/TokenController
```

Este comando creará el controlador `app/Http/Controllers/API/TokenController.php`, al que le debemos añadir un método de creación de _tokens_ y un método de eliminación de _tokens_. De esta forma, el contenido de este controlador será:

```php
<?php
// in app/Http/Controllers/API/TokenController.php
namespace App\Http\Controllers\API;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Validation\ValidationException;
use App\Models\User;
use Illuminate\Support\Facades\Hash;

class TokenController extends Controller
{
    /**
     * Store a newly created personal access token.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $request->validate([
            'email' => 'required|email',
            'password' => 'required'
        ]);

        // comprobamos si existe un usuario con ese email
        $user = User::where('email', $request->email)->first();

        // si no existe o la contraseña no es correcta, devolvemos un error
        if (! $user || ! Hash::check($request->password, $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        // si el usuario existe y la contraseña es correcta, generamos un token
        return response()->json([
            'token_type' => 'Bearer',
            'access_token' => $user->createToken('token_name')->plainTextToken // token name you can choose for your self or leave blank if you like to
        ]);
    }

    /**
     * Delete the user's personal access token.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function destroy(Request $request)
    {
        $request->user()->currentAccessToken()->delete();
    }

}
```

### Rutas de tokens

También debemos crear las rutas necesarias para poder emitir y borrar los _personal access tokens_.


```php
// in routes/api.php
Route::prefix('v1')->group(function () {
    Route::middleware('auth:sanctum')->group(function () {
        Route::get('/user', function (Request $request) {
            $user = $request->user();
            $user->fullName = $user->nombre . ' ' . $user->apellidos;
            return $user;
        });

        Route::apiResource('ciclos', CicloController::class);
    });

    // emite un nuevo token
    Route::post('tokens', [TokenController::class, 'store']);
    // elimina el token del usuario autenticado
    Route::delete('tokens', [TokenController::class, 'destroy'])->middleware('auth:sanctum');

});

```

Debemos acordarnos de incluir la ruta al controlador de tokens

```php
use App\Http\Controllers\API\TokenController;
```    

Para proteger las rutas, hemos indicado que se utilice el `guard sanctum`:

Debemos fijarnos que la respuesta a la ruta `GET /api/v1/user` la hemos modificado para que incluya un atributo `fullName`, con el nombre completo del usuario que está autenticado, que es el que espera recibir _react-admin_.

A partir de ese momento, para acceder a esa ruta deberemos utilizar los _tokens_ generados por _Sanctum_. Para ello, a cada petición le debemos añadir un `Header` de tipo `Authorization` cuyo contenido será `Bearer + [el token recibido]`.

Por ejemplo:

```http
Authorization                   Bearer 6|i5q80n6dtqP4QFdbvfz110myF8LhyeuNB00wUwK6
```

> Debemos eliminar la protección que teníamos sobre la ruta `dashboard` en el fichero `routes/web.php`. Una vez que en esa ruta se cargue la aplicación de React-admin, será a través de esta aplicación desde donde se gestionará la autenticación, incluida la pantalla de _login_.
```php
Route::get('/dashboard', function () {
    return Inertia::render('Dashboard');
})->name('dashboard');
```

## Frontend

En el frontend, debemos modificar el `authProvider` para que utilice el token de autenticación en las peticiones a la API y el `dataProvider` para que incluya el token en las cabeceras de las peticiones.

Sin detenernos en el detalle de la implementación, el código de estos dos ficheros será el siguiente:

### AuthProvider

```php
// in resources/js/react_admin/authProvider.js

import { dataProvider } from "./dataProvider";

export const authProvider = {
    login: ({ username, password }) => {
        return dataProvider.createToken( username, password )
        .then(response => {
            if (response.status < 200 || response.status >= 300) {
              throw new Error(response.statusText);
            }
            localStorage.setItem('auth', JSON.stringify(response.json));
        })
        .catch((e) => {
                throw new Error('Network error')
        });
    },
    logout: () => {
        let token = localStorage.getItem('auth')
        if (token) {
            token = JSON.parse(localStorage.getItem('auth'))
            localStorage.removeItem('auth');
            return dataProvider.deleteToken
                .then(() => ('login'))
                .catch((error) => {
                    throw error
                });
        } else {
            return Promise.resolve()
        }
    },
    checkAuth: () =>
        (localStorage.getItem('auth') ? Promise.resolve() : Promise.reject()),
    checkError: (error) => {
        const status = error.status;
        if (status === 401 || status === 403) {
            localStorage.removeItem('auth');
            return Promise.reject();
        }
        // other error code (404, 500, etc): no need to log out
        return Promise.resolve();
    },
    getIdentity: () => {
        const token = localStorage.getItem('auth') ? JSON.parse(localStorage.getItem('auth')) : undefined
        if (!token) {
            throw new Error('No auth token');
        }

        return dataProvider.getIdentity()
            .then(( data ) => {
                return data.json
            })
            .catch(() => {
                throw new Error('Network error')
            });
    },
    getPermissions: () => Promise.resolve('')
};

```

## DataProvider

Nuestro _authProvider_ utiliza un _dataProvider_ basado en [JSON-Server](https://github.com/typicode/json-server), al que le añadiremos el **token de autenticación** en las cabeceras de las peticiones, siempre y cuando este _token_ exista en el `localStorage` del cliente, y funciones para solicitar a la API las peticiones relacionadas con el `login()`, `logout()` y `getIdentity()`.

El código para el dataProvider es el siguiente:

```php
// in resources/js/react-admin/dataProvider.ts
import { fetchUtils } from 'react-admin';
import jsonServerProvider from "ra-data-json-server";

const httpClient = (url, options = {}) => {
    if (!options.headers) {
        options.headers = new Headers({ Accept: 'application/json' });
    }
    const token = localStorage.getItem('auth') ? JSON.parse(localStorage.getItem('auth')) : undefined
    if (token) {
        options.headers.set('Authorization', `${token.token_type} ${token.access_token}`);
    }
    return fetchUtils.fetchJson(url, options);
};

const dataProvider = jsonServerProvider(
    import.meta.env.VITE_JSON_SERVER_URL,
    httpClient
);

const url = `${import.meta.env.VITE_JSON_SERVER_URL}`;

dataProvider.createToken = (email, password) => {
    return httpClient(url + '/tokens', {
        method: 'POST',
        body: JSON.stringify({ email, password }),
        headers: new Headers({ 'Content-Type': 'application/json' }),
    });
};

dataProvider.deleteToken = () => {
    return httpClient(url + '/tokens', {
        method: 'DELETE',
        headers: new Headers({ 'Content-Type': 'application/json' }),
    });
};

dataProvider.getIdentity = () => {
    return httpClient(url + '/user', {
        method: 'GET',
        headers: new Headers({ 'Content-Type': 'application/json' }),
    });
};

export { dataProvider };

```

## Pruebas

### Rutas de las peticiones y variables de entorno.

Las rutas configuradas  en el fichero `routes/api.php` en el apartado [Rutas de tokens](#rutas-de-tokens) las hemos configurado para que se utilice la versión `v1` de la API. Por tanto, las peticiones deberán dirigirse a `http://marcapersonalfp.test/api/v1/...` y, para ello, debemos modificar la variable de entorno `VITE_JSON_SERVER_URL` en el fichero `.env`:

```bash
VITE_JSON_SERVER_URL=http://marcapersonalfp.test/api/v1
```

### Prueba de la autenticación

Antes de realizar una prueba debemos asegurarnos que no tenemos ninguna cookie de sesión activa ni ningún token. Para ello, accedemos a las herramientas de desarrollador del navegador (_F12_ o _Ctrl+Shift+I_) y **borramos las cookies y el almacenamiento local** (`localStorage`).

A continuación:

- Accedemos a la _URL_ [http://marcapersonalfp.test/dashboard](http://marcapersonalfp.test/dashboard)
- Como la ruta está protegida, nos redirige a la ruta `/login`. El formulario de _login_ que nos presenta es el que se instaló con _breeze_ y sirve para autenticarnos con el guard `web` y, por lo tanto, para autenticarnos en todas las rutas definidas en el fichero `routes/web.php`.
- Si volvemos a intentar acceder a la _URL_ [http://marcapersonalfp.test/dashboard](http://marcapersonalfp.test/dashboard), en esta ocasión comienza a presentarnos el _Dashboard_ pero, casi inmediatamente nos muestra el formulario de _login_ de _React-admin_. Esto ocurre porque _React-admin_ realiza peticiones a la API, en las que tenemos definido el guard `sanctum`, es decir, por tokens.
- Si nos autenticamos con el formulario de _login_ de _React-admin_, almacenará el token el `localStorage` y se nos presentará el _Dashboard_ de _React-admin_.
