# Reutilizar plantillas

En internet, podemos encontrar sitios que ofrecen plantillas gratuitas para utilizar en nuestros proyectos. Por ejemplo, en [HTML5 UP!](https://html5up.net/) podemos encontrar una gran cantidad de plantillas gratuitas para utilizar en nuestros proyectos. En este apartado, vamos a utilizar una de estas plantillas para crear el _layout_ principal de nuestra web.

## Descargar plantilla

De entre las muchas plantillas existentes, vamos a utilizar la plantilla [Dopetrope](https://html5up.net/dopetrope). Para descargarla, vamos a la página de la plantilla y pulsamos el botón de descarga o hacemos click en el siguiente enlace directo: [Dopetrope](https://html5up.net/dopetrope/download).

Crearemos una carpeta llamada `dopetrope` dentro de la carpeta `public` y descomprimiremos el contenido de la plantilla dentro de esta carpeta.

## Crear el layout principal con el index de _dopetrope_

También crearemos una carpeta llamada `dopetrope` dentro de la carpeta `resources/views` y dentro de esta carpeta trasladaremos los archivos html de la plantilla que hemos descargado y que hemos colocado inicialmente en `public\dopetrope`. En este caso, vamos a utilizar el archivo `index.html` como _layout_ principal de nuestra web.

Si queremos utilizar _Blade_ en estos archivos, debemos cambiar su extensión a `.blade.php`. En el caso, de `index.html` vamos a cambiar el nombre completo a `master.blade.php`.

## Utilizar el layout de dopetrope

Para utilizar el _layout_ de dopetrope, vamos a modificar el archivo `resources/views/layouts/master.blade.php` haciendo que extienda del _layout_ de dopetrope. Para ello, vamos a modificar la primera línea del archivo de la siguiente forma:

```
@extends('dopetrope.master')
```

Con este simple cambio, hemos provocado que todas las vistas actuales extiendan de `master.blade.php` y éste, a su vez, del _layout_ de dopetrope, por lo que si probamos la web en el navegador, veremos que se muestra el _layout_ de dopetrope con el contenido de ejemplo que hemos definido en cada una de las vistas.

Hacer esto no es suficiente, ya que el _layout_ de dopetrope utiliza una serie de _assets_ (imágenes, hojas de estilo, javascript, etc.) que hemos desplazado a la carpeta `public` de nuestro proyecto. Por lo tanto, debemos modificar las rutas de estos _assets_ para que apunten a la carpeta `public` de nuestro proyecto. Para ello, vamos a modificar el archivo `resources/views/dopetrope/master.blade.php` de la siguiente forma:

- En la línea 12, cambiamos la ruta de la hoja de estilos de la siguiente forma:

```
<link rel="stylesheet" href="{{ asset('/dopetrope/assets/css/main.css') }}" />
```

- En las líneas 384 y siguientes, cambiamos las rutas de los _javascript_ para que los busque en la carpeta `dopetrope` de `public`:

```
<script src="{{ asset('/dopetrope/assets/js/jquery.min.js') }}"></script>
<script src="{{ asset('/dopetrope/assets/js/jquery.dropotron.min.js') }}"></script>
<script src="{{ asset('/dopetrope/assets/js/browser.min.js') }}"></script>
<script src="{{ asset('/dopetrope/assets/js/breakpoints.min.js') }}"></script>
<script src="{{ asset('/dopetrope/assets/js/util.js') }}"></script>
<script src="{{ asset('/dopetrope/assets/js/main.js') }}"></script>
```

- En la línea 21, cambiaremos también la ruta de la imagen del logo de la siguiente forma (tendremos que redefinir `APP_URL` en el fichero `.env`):

```
<h1>
    <a href="{{ url(env('APP_URL', 'http://marcapersonalFP.test')) }}">
        <img src="{{ asset('/images/mp-logo.png') }}" alt="Logo Marca Personal FP" width="200px"/>
    </a>
</h1>
```

- Las referencias a cada una de las imágenes también habrá que cambiarlas para que apunten a la carpeta `dopetrope\images` de `public`. Para remplazar, desde Visual Studio Code, todas las ocurrencias que incluyan el texto `images/pic`, seguidas de un número y el texto `.jpg` por "{{ asset('/dopetrope/images/pic" seguidos del número anterior y `.jpg') }}`, necesitamos utilizar una búsqueda con expresión regular, en el que el texto de la búsqueda sea el siguiente `src="images/pic([0-9]*).jpg"` y el texto para remplazar sería `src="{{ asset('/dopetrope/images/pic$1.jpg') }}"`

## Distribuir el contenido del layout en partes

En el _layout_ de dopetrope, el contenido de la web se divide en tres secciones (`section`):

- `header`: cabecera de la web
- `main`: contenido principal de la web
- `footer`: pie de página de la web

Vamos a crear _partials_ con el contenido de cada una de estas partes y las vamos a incluir en el _layout_ principal.

### Crear partials

En primer lugar, vamos a crear los _partials_ que vamos a utilizar para cada una de las partes de la web. Para ello, vamos a crear una carpeta llamada `partials` dentro de la carpeta `resources/views/dopetrope` y dentro de esta carpeta vamos a crear (podemos hacerlo con `artisan`) los siguientes archivos:

- `header.blade.php`: cabecera de la web
- `main.blade.php`: contenido principal de la web
- `footer.blade.php`: pie de página de la web

En cada uno de estos archivos, vamos a mover el contenido de la sección correspondiente de `master.blade.php`.

### Incluir partials en el layout

Desde el archivo `dopetrope/master.blade.php` referenciaremos a cada partial creado con el `@include` correspondiente, por lo que todo el código desplazado a esos archivos será sustituido por:

```
@include('dopetrope.partials.header')
@include('dopetrope.partials.main')
@include('dopetrope.partials.footer')
```

## Incluir la sección `content`

En la plantilla master.blade.php teníamos una sección `content`, que era el lugar en el que las vistas colocaban el contenido propio de la acción que se había solicitado.

Esto se hacía incorporando `@yield('content')` a la plantilla. En este caso, incorporaremos el código que se muestra a continuación en el archivo `dopetrope/partials/main.blade.php`, justo antes de la línea que continene el comentario `<!-- Portfolio -->`:

```
                            <!-- content -->
                            <section>
                                <header class="major">
                                    <h2>Content</h2>
                                </header>
                                <div class="row">
                                    <div class="container">
                                        @yield('content')
                                    </div>
                                </div>
                            </section>
```

## incluir el `navbar`

En el _partial_ `header.blade.php` vamos a sustituir los códigos que hay bajo los comentarios `<!-- Logo -->` y `<!-- Nav -->` por `@include('partials.navbar')`.

Para que se vea correctamente, vamos a incluir en `resources/views/dopetrope/layouts/master.blade.php` las referencias a los archivos _css_ y _javascript_ que utiliza el _navbar_ y que estaban en `resources/views/layouts/master.blade.php`:

- En la línea 12 vamos a incluir la referencia a la hoja de estilos del _navbar_ (_desplazando la línea que comienza por `<link rel="stylesheet" href="{{ asset('/d...`_ a la línea 13):

```
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN" crossorigin="anonymous">
```

- En la línea 37 (_tras las referencias a los scripts `<script src="{{ asset('/d...`_) vamos a incluir la referencia al _javascript_ del _navbar_:

```
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL" crossorigin="anonymous"></script>
```

