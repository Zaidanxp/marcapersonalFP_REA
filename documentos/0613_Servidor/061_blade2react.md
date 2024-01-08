# De Blade a React
En los capítulos anteriores, hemos utilizado el gestor de plantillas _Blade_ para visualizar las vistas de nuestra página web. A partir de ahora, vamos a integrar contenidos del **módulo de Cliente** y del **módulo de Servidor**, para crear una **API REST** con _Laravel_ y consumirla desde una aplicación _React_.

## Reinstalar Breeze

Volveremos a ejecutar el comando de instalación de los recursos que utiliza _Breeze_:

```bash
php artisan breeze:install
```

Pero, en esta ocasión, elegiremos como framework **React with Inertia**:

```
    ┌ Which Breeze stack would you like to install? ───────────────┐
    │   ○ Blade with Alpine                                           │
    │   ○ Livewire (Volt Class API) with Alpine                       │
    │   ○ Livewire (Volt Functional API) with Alpine                  │
    │ › ● React with Inertia                                          │
    │   ○ Vue with Inertia                                            │
    │   ○ API only                                                    │
    └───────────────────────────────────────────────────────┘
```

No elegiremos ninguna de las características opcionales::

```
 ┌ Would you like any optional features? ───────────────────────┐
 │ › ◻ Dark mode                                                    │
 │   ◻ Inertia SSR                                                  │
 │   ◻ TypeScript (experimental)                                    │
 └────────────────────────────────────────────────────────┘
```

Mantenemos, eso sí, la elección de PHPUnit para ejecución de los tests:

```
 ┌ Which testing framework do you prefer? ──────────────────────┐
 │ › ● PHPUnit                                                      │
 │   ○ Pest                                                         │
 └────────────────────────────────────────────────────────┘
```

## Recuperar las rutas

Como en la ejecución anterior de _Breeze_, necesitamos recuperar las rutas que ya teníamos. En esta ocasión, en lugar de tener que jugar con `git` para conseguir ese objetivo, ganaremos tiempo, sustituyendo el contenido del archivo `routes/web.php` por [esta versión del fichero](./materiales/ejercicios-laravel/routes_web_inertia.php).

## Recuperar las vistas _Auth_

Anteriormente, habíamos modificado algunas vistas Auth para que incluyeran información necesaria para nuestra aplicación:

- `resources/views/auth/register.blade.php`
- `resources/views/profile/edit.blade.php`
- `resources/views/profile/partials/information-insignias.blade.php`
- `resources/views/profile/partials/update-avatar-form.blade.php`
- `resources/views/profile/partials/update-curriculo-information-form.blade.php`
- `resources/views/profile/partials/update-profile-information-form.blade.php`

Al utilizar Inertia-React para visualizar la información, estas vistas han cambiado, incluso de carpeta.

La vista `resources/views/auth/register.blade.php` la tenemos ahora en `resources/js/Pages/Auth/Register.jsx`, mientras que las vistas relativas al _profile_ están en `resources/js/Pages/Profile/Edit.jsx` y los _partials_ en `resources/js/Pages/Profile/Partials`.
