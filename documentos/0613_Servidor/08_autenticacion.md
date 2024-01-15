# Autenticación

## Configuración

El archivo de configuración de autenticación de su aplicación se encuentra en config/auth.php. Este archivo contiene varias opciones bien documentadas para modificar el comportamiento de los servicios de autenticación de Laravel.


En ese archivo se puede observar la diferente configuración entre el guard `web` y `api`

```php
    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
        'api' => [
            'driver' => 'token',
            'provider' => 'users',
            'hash' => false,
        ],
    ],
```

## Consideraciones de la base de datos

De forma predeterminada, _Laravel_ incluye el modelo _Eloquent_ `App\Models\User`  en su directorio `app/Models`. Este modelo se puede utilizar con el controlador de autenticación predeterminado de _Eloquent_.

## Tipos de autenticación

- Cookie/Sesión:
    - Uso de formulario web para envío de credenciales (nombre de usuario y contraseña).
    - Almacenamiento del usuario autenticado en la sesión (servidor).
    - El servidor envía una cookie con la ID de sesión.
    - Con cada petición, el cliente envía la cookie con el ID de sesión.
    - El servidor recupera la sesión según ese ID y el usuario asociado a la sesión.
- Token: Cuando un cliente necesita autenticarse para acceder a una API, las cookies no se utilizan normalmente para la autenticación porque no hay un navegador web:
    - El cliente envía un token de API a la API en cada solicitud.
    - El servidor valida el token entrante contra una tabla de tokens API válidos
    - Autentica la solicitud como realizada por el usuario asociado con ese token API.

## Kits de autenticación.

_Laravel_ facilita paquetes gratuitos para incorporar la mayoría de los requisitos de autenticación a nuestras aplicaciones:

- [Laravel Breeze](https://laravel.com/docs/starter-kits#laravel-breeze)
- [Laravel Jetstream](https://jetstream.laravel.com/)
- [Laravel Fortify](https://laravel.com/docs/fortify)

No obstante, el uso de los paquetes anteriores, no entra dentro del alcance de este módulo.

## Servicios de autenticación de API

Para gestionar los _tokens_ que se transmitirán entre **cliente** y **servidor**, Laravel proporciona dos paquetes:

- _Passport_: utiliza _OAuth2_, para autenticar al usuario a través de servidores de autenticación externos (_Google_, _Facebook_, _Twitter_,...)
- _Sanctum_: un paquete de **autenticación híbrido web / API**, que puede administrar todo el proceso de autenticación de su aplicación.

Por su mayor sencillez, utilizaremos _Sanctum_ para nuestro desarrollo y dejaremos _Passport_ como **ampliación**.

## Resumen y elección de su pila

En resumen, si vamos a acceder a la aplicación utilizando un navegador y vamos a crear una aplicación Laravel monolítica, su aplicación utilizará los servicios de autenticación integrados de Laravel.

En cambio, si nuestra aplicación ofrece una API que será consumida por terceros, elegiremos entre _Passport_ o _Sanctum_ para proporcionar autenticación de _token API_ para la aplicación.

En general, se debe preferir _Sanctum_ cuando sea posible, ya que es una solución simple y completa para la **autenticación de API**, **autenticación de SPA** y **autenticación móvil**. Al usar _Sanctum_, deberá implementar manualmente sus propias rutas de autenticación de backend o utilizar _Laravel Breeze_ como un servicio de backend de autenticación que proporciona rutas y controladores para funciones como registro, restablecimiento de contraseña, verificación de correo electrónico y más.

Se puede elegir _Passport_ cuando su aplicación necesita absolutamente todas las características proporcionadas por la especificación **OAuth2**.
