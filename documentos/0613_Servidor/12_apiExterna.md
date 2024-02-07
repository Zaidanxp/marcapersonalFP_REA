# Consumiendo API externas

## Introducción

_Laravel_ proporciona una API expresiva y mínima alrededor del [_Guzzle HTTP client_](http://docs.guzzlephp.org/en/stable/), permitiéndote realizar rápidamente peticiones _HTTP_ salientes para comunicarte con otras aplicaciones web. El envoltorio de _Laravel_ alrededor de _Guzzle_ se centra en sus casos de uso más comunes y una maravillosa experiencia de desarrollo.

Antes de comenzar, debes asegurarte de haber instalado el paquete _Guzzle_ como una dependencia de tu aplicación. Por defecto, _Laravel_ incluye automáticamente esta dependencia. Sin embargo, si previamente has eliminado el paquete, puedes instalarlo de nuevo a través de Composer:

```bash
composer require guzzlehttp/guzzle
```

## _API_ abiertas

Podéis encontrar un listado muy completo de API accesibles gratuítamente en [este repositorio de GitHub](https://github.com/public-api-lists/public-api-lists).

En ese listado, podrás encontrar API de todo tipo, que seguro te vendrán bien para el proyecto que vayas a iniciar en breve.

## API de GitHub

Para la finalidad de nuestro proyecto, vamos a utilizar la API de GitHub para almacenar los proyectos de los estudiantes. La API de GitHub es una **API RESTful** que se puede utilizar para acceder a los recursos de GitHub.

### Autenticación

Para comenzar a utilizar la API de GitHub, necesitas obtener un token de acceso. Puedes obtener un token de acceso siguiendo los pasos que se indican en la [documentación de GitHub](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token).

Para no tener limitaciones de seguridad, vamos a utilizar un token de acceso personal. Para ello, sigue los siguientes pasos:

1. En la esquina superior derecha de cualquier página, haz clic en tu foto de perfil y, a continuación, haz clic en **Settings**.
2. En la barra lateral izquierda, haz clic en **Developer settings**.
3. En la barra lateral izquierda, haz clic en **Personal access tokens**.
4. Haz clic en **Generate new token**.
5. Escribe tu contraseña de GitHub.
6. En **Note**, escribe un nombre para tu token.
7. Selecciona los ámbitos, o permisos, que deseas otorgar a este token. Para nuestro caso, selecciona **repo**.
8. Haz clic en **Generate token**.
9. Copia el token generado.
10. Guarda el token en el archivo `.env` de tu proyecto asociado a la variable `GITHUB_TOKEN`.
11. Crea también la siguiente variable en el archivo `.env.example`:

    ```bash
    GITHUB_TOKEN="Write your personal access token here"
    ```
12. Aprovechamos para crear una nueva variable de entorno en los fichero `.env` y `.env.example`:

    ```bash
    GITHUB_OWNER="The name of your user or organization in GitHub"
    ```

### Adaptar la base de datos

En la creación de un nuevo repositorio, la API de GitHub devuelve la url del nuevo repositorio. Vamos a aprovechar esta url para almacenarla en la base de datos, asociada al proyecto correspondiente.

Para ello, vamos a modificar la tabla `proyectos` para añadir un nuevo campo `url_github` que almacenará la url del repositorio en GitHub.

```bash
php artisan make:migration add_url_github_to_proyectos_table --table=proyectos
```

Este comando creará un nuevo archivo en el directorio `database/migrations` de tu aplicación. En este archivo, vamos a definir los campos que vamos a añadir a la tabla `proyectos`.

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::table('proyectos', function (Blueprint $table) {
            $table->string('url_github')->nullable();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::table('proyectos', function (Blueprint $table) {
            $table->dropColumn('url_github');
        });
    }
}
```

El atributo recién creado `url_github` lo incluiremos también entre los campos del array `$fillable` del modelo `Proyecto`.

```diff
         'metadatos',
         'calificacion',
-         'fichero'
+         'fichero',
+         'url_github',
     ];
```

> Ejecuta después el comando `php artisan migrate` para aplicar los cambios en la base de datos.

### Service Provider para la API de GitHub

Las peticiones a la _API_ de _GitHub_ las vamos a centralizar en un **_Service Provider_** de _Laravel_.

#### Crear el _Service Provider_

Para la creación del _Service Provider_, vamos a utilizar el siguiente comando _Artisan_:

```bash
php artisan make:provider GitHubServiceProvider
```

Este comando creará un nuevo archivo en el directorio `app/Providers` de tu aplicación. En este archivo, vamos a definir los métodos que se encarguarán de realizar las peticiones a la _API_ de _GitHub_.

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class GitHubServiceProvider extends ServiceProvider
{
    /**
     * Register services.
     */
    public function register(): void
    {
        //
    }

    /**
     * Bootstrap services.
     */
    public function boot(): void
    {
        //
    }
}
```

#### Configuración inicial del cliente _Guzzle_

Para comenzar a realizar peticiones a la _API_ de _GitHub_, vamos a configurar el cliente _Guzzle_ en el constructor del _Service Provider_.

```diff
 namespace App\Providers;
 
 use Illuminate\Support\ServiceProvider;
+use GuzzleHttp\Client;
 
 class GitHubServiceProvider extends ServiceProvider
 {
+    const GITHUB_API_VERSION = '2022-11-28';
+    const GITHUB_API_ENDPOINT = 'https://api.github.com';
+    protected $client;
+
+    public function __construct()
+    {
+        $this->client = new Client([
+            'base_uri' => self::GITHUB_API_ENDPOINT,
+            'headers' => [
+                'Accept' => 'application/vnd.github+json',
+                'Authorization' => 'Bearer ' . env('GITHUB_TOKEN'),
+                'X-GitHub-Api-Version' => self::GITHUB_API_VERSION,
+            ]
+        ]);
+    }
+
     /**
      * Register services.
```

En el código anterior hemos definido los parámetros de la cabecera que se utilizarán en todas las peticiones a la _API_ de _GitHub_, en especial, el token de autenticación que hemos guardado en el archivo `.env`.

#### Crear un nuevo repositorio

Para crear un nuevo repositorio en GitHub, vamos a utilizar el servicio que hemos definido en el Service Provider, al que le vamos a aádir un método `createRepo(Proyecto $proyecto)`, que recibirá una instancia del modelo `Proyecto` y que se encargará de realizar la petición a la API de GitHub para crear un nuevo repositorio.

```diff
    public function __construct()
    ...
         ]);
     }
 
+    public function createRepo(Proyecto $proyecto)
+    {
+        $owner = env('GITHUB_OWNER');
+        $githubResponse = $this->client->post("/orgs/{$owner}/repos", [
+            'json' => $proyecto->getGithubSettings()
+        ]);
+
+        if($githubResponse->getStatusCode() === 201) {
+            $githubResponse = $this->client->get($githubResponse->getHeader('Location')[0]);
+        }
+        return $githubResponse;
+    }
+
     /**
      * Register services.
```

Para que este método funcione, necesitamos que el modelo `Proyecto` tenga un método `getGithubSettings()` que devuelva un array con los parámetros necesarios para crear un nuevo repositorio en GitHub.

```diff
    public static function contarProyectos()
    {
        $proyectos = self::all()->count();
         return $proyectos;
    }
 
+    public function getGithubSettings() {
+        // TODO obtener un nombre según curso, familia, ciclo, nombre
+        return [
+            'org' => env('GITHUB_OWNER'),
+            'name' => $this->nombre,
+            'description' => $this->metadatos,
+            'homepage' => $this->url_github,
+            'private' => false,
+            'has_issues' => true,
+            'has_projects' => false,
+            'has_wiki' => false,
+        ];
+    }
+
     public function ciclos(): BelongsToMany
```

#### Borrar el repositorio

Para borrar un repositorio en GitHub, vamos a utilizar el servicio que hemos definido en el Service Provider, al que le vamos a aádir un método `deleteRepo(Proyecto $proyecto)`, que recibirá una instancia del modelo `Proyecto` y que se encargará de realizar la petición a la API de GitHub para borrar el repositorio.

```diff
         return $githubResponse;
     }
 
+    public function deleteRepo(Proyecto $proyecto)
+    {
+        $owner = env('GITHUB_OWNER');
+        $repo = $proyecto->getRepoNameFromURL();
+        return $this->client->delete("/repos/{$owner}/{$repo}");
+    }
+
     /**
      * Register services.
```

Para que este método funcione, necesitamos que el modelo `Proyecto` tenga un método `getRepoNameFromURL()` que devuelva el nombre del repositorio en GitHub.

```diff
     public function getGithubSettings() {
        ...
         ];
     }
 
+    public function getRepoNameFromURL() {
+        $url = $this->url_github;
+        $repoName = substr($url, strripos($url, '/') + 1);
+        return $repoName;
+    }
+
     public function ciclos(): BelongsToMany
     {
```

#### Enviar los ficheros del proyecto

El envío de los ficheros del proyecto conlleva varios pasos:

- Descomprimir el fichero en el servidor.
- Obtener el `sha` de cada fichero en GitHub.
- Enviar cada uno de los ficheros al repositorio en GitHub.

La realización de los anteriores pasos requiere añadir los siguientes métodos al _Service Provider_:

```php
    public function pushZipFiles(Proyecto $proyecto)
    {
        $tmpdir = sys_get_temp_dir() . DIRECTORY_SEPARATOR . $proyecto->getRepoNameFromURL();
        $this->unzipFiles($proyecto, $tmpdir);
        $files = collect(File::allFiles($tmpdir));
        $files->each(function ($file, $key) use ($proyecto) {
            $this->sendFile($proyecto, $file);
        });

        File::deleteDirectory($tmpdir);
    }

    public function unzipFiles(Proyecto $proyecto, $tmpdir)
    {
        $zip = new ZipArchive;
        $zipPath = storage_path($proyecto->fichero);
        $zip->open($zipPath);
        $zip->extractTo($tmpdir);
        $zip->close();
    }

    public function getShaFile($repoName, $file)
    {
        $owner = env('GITHUB_OWNER');
        $path = $file->getRelativePathname();
        $response = $this->client->get("/repos/{$owner}/{$repoName}/contents/{$path}");
        return $response->json()['sha'];
    }

    public function sendFile(Proyecto $proyecto, $file)
    {
        $repoName = $proyecto->getRepoNameFromURL();
        $owner = env('GITHUB_OWNER');
        $path = $file->getRelativePathname();
        $sha = $this->getShaFile($repoName, $file);
        $response = $this->client->put("/repos/{$owner}/{$repoName}/contents/{$path}", [
            'json' => [
                'message' => 'Add ' . $file->getRelativePathname(),
                'content' => base64_encode(file_get_contents($file->getRealPath())),
                'sha' => $sha
            ]
        ]);
        return $response;
    }
```

> Para la ejecución de los métodos anteriores, se deberán añadir las siguientes directivas `use` al principio del archivo del Service Provider:

```php
use App\Models\Proyecto;
use Illuminate\Support\Facades\File;
use ZipArchive;
```

### Cambios en el controlador

Para poder utilizar el _Service Provider_ que hemos creado, vamos a modificar el controlador `ProyectoController` para que utilice el _Service Provider_, después de la subida del fichero en el método `update`:
