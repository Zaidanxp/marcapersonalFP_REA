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

# Instalar React-admin

## Crear la aplicación React-admin

Preferiblemente desde una carpeta diferente a la de nuestro proyecto _Laravel_, ejecutaremos el siguiente comando:

```bash
npm init react-admin mpfpra
```

y haremos las siguientes elecciones:

```
  Fakerest
  A client-side, in-memory data provider that use a JSON object as its initial data.

❯ JSON Server
  A data provider based on the JSON Server API (https://github.com/typicode/json-server)

  Simple REST
  A Simple REST data provider (https://github.com/marmelab/react-admin/tree/master/packages/ra-data-simple-rest)

  None
  I'll configure the data provider myself.
```

```
Select the auth provider you want to use, and validate with Enter:
❯ Hard coded local username/password
  Hard coded local username/password

  None
  No authProvider
```

```
Enter the name of a resource you want to add, and validate with Enter (leave empty to finish):
users 
```

```
How do you want to install the dependencies?
  Using npm

  Using yarn

❯ Don't install dependencies, I'll do it myself.
```

## Trasladar el código a _Laravel_

El traslado del código fuente se realizará en varios pasos:

- Añadir a los archivos .env y .env.example la variable de entorno
    
    ```VITE_JSON_SERVER_URL=http://marcapersonalfp.test/api/records```

- Mezclar el fichero `mpfpra/package.json` con el existente en la raíz de nuestro proyecto. El resultado debería ser similar al que se muestra a continuación:
    ```json
    {
        "name": "MarcaPersonalFP",
        "private": true,
        "type": "module",
        "scripts": {
            "dev": "vite",
            "build": "vite build",
            "serve": "vite preview",
            "type-check": "tsc --noEmit",
            "lint": "eslint --fix --ext .js,.jsx,.ts,.tsx ./src",
            "format": "prettier --write ./src"
        },
        "dependencies": {
            "ra-data-json-server": "^4.16.0",
            "react": "^18.2.0",
            "react-admin": "^4.16.0",
            "react-dom": "^18.2.0"
        },
        "devDependencies": {
            "@headlessui/react": "^1.4.2",
            "@inertiajs/react": "^1.0.0",
            "@tailwindcss/forms": "^0.5.3",
            "@typescript-eslint/parser": "^5.60.1",
            "@typescript-eslint/eslint-plugin": "^5.60.1",
            "@types/node": "^18.16.1",
            "@types/react": "^18.0.22",
            "@types/react-dom": "^18.0.7",
            "@vitejs/plugin-react": "^4.0.1",
            "alpinejs": "^3.4.2",
            "autoprefixer": "^10.4.12",
            "axios": "^1.1.2",
            "eslint": "^8.43.0",
            "eslint-config-prettier": "^8.8.0",
            "eslint-plugin-react": "^7.32.2",
            "eslint-plugin-react-hooks": "^4.6.0",
            "laravel-vite-plugin": "^0.8.0",
            "postcss": "^8.4.18",
            "prettier": "^2.8.8",
            "tailwindcss": "^3.2.1",
            "typescript": "^5.1.6",
            "vite": "^4.3.9"
        }
    }
    ```
- Copiar el fichero `mpfpra/public/manifest.json` a la carpeta `public` de nuestro proyecto.
- Copiar la carpeta `mpfpra/src` a la carpeta `resources/js` y cambiarle el nombre a `react-admin`.
- Copiar el fichero `mpfpra/tsconfig.json` a la carpeta raíz del proyecto.
- Mezclar el fichero `mpfpra/vite.config.js` con el existente en la raíz de nuestro proyecto. El resultado debería ser similar al que se muestra a continuación:
    ```javascript
    import { defineConfig } from 'vite';
    import laravel from 'laravel-vite-plugin';
    import react from '@vitejs/plugin-react';

    export default defineConfig({
        plugins: [
            laravel({
                input: 'resources/js/app.jsx',
                refresh: true,
            }),
            react(),
        ],
        define: {
            'process.env': process.env,
        },
        server: {
            host: true,
        },
        base: './',
    });

    ```
 - Para responder a la petición `/dashboard`, modificaremos el código del archivo `resources/js/Pages/Dashboard.jsx` con el siguiente contenido:
    ```javascript
    // in src/index.tsx
    import React from "react";
    import { App } from "../react-admin/App";

    const Dashboard = () => {
        return(
            <React.StrictMode>
                <App />
            </React.StrictMode>
        );
    }

    export default Dashboard;
    ```
- Finalmente, ejecutaremos `npm install` para instalar las dependencias. y `npm run dev` para arrancar el servidor de desarrollo.
