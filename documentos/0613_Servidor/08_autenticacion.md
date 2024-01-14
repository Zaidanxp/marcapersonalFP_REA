# Autenticación

## Configuración

En `config/auth.php`

```php
        'api' => [
            'driver' => 'token',
            'provider' => 'users',
            'hash' => false,
        ],
```

## Rutas

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

## TokenController

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

        $user = User::where('email', $request->email)->first();

        if (! $user || ! Hash::check($request->password, $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

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


## AuthProvider

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
