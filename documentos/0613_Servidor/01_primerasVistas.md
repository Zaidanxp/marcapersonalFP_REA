## Layout principal de las vistas con Bootstrap

En este ejercicio debéis esbozar el _layout_ base que van a utilizar el resto de vistas del sitio web.

Para desarrollar el _layout_ principal de nuestro sitio, creamos el fichero `resources/views/layouts/master.blade.php` y pondremos en él todo el contenido que se deba repetir en todas las páginas de nuestra aplicación.

## Crear el resto de vistas

En este ejercicio vamos terminar una primera versión estable de la web. En primer lugar crearemos las vistas asociadas a cada ruta, las cuales tendrán que extender del _layout_ que hemos hecho en el ejercicio anterior y mostrar (en la sección _content_ del _layout_) el texto de ejemplo que habíamos definido para cada ruta en el ejercicio anterior. En general, todas las vistas tendrán un código similar al siguiente (variando únicamente la sección _content_):
```
@extends('layouts.master')

@section('content')

    Pantalla principal

@stop
```

Para organizar mejor las vistas las vamos a agrupar en sub-carpetas dentro de la carpeta `resources/views` siguiendo la siguiente estructura:

Vista | Carpeta | Ruta asociada
------|---------|--------------
home.blade.php | resources/views/ | /
login.blade.php | resources/views/auth/ | login
index.blade.php | resources/views/proyectos/ | proyectos
show.blade.php | resources/views/proyectos/ | proyectos/show/{id}
create.blade.php | resources/views/proyectos/ | proyectos/create
edit.blade.php | resources/views/proyectos/ | proyectos/edit/{id}

Crearemos una vista separada para todas las rutas excepto para la ruta `logout`, la cual no tendrá ninguna vista asociada.

Por último vamos a actualizar las rutas del fichero `routes/web.php` para que se carguen las vistas que acabamos de crear. Acordaos que para referenciar las vistas que están dentro de carpetas la barra `/` de separación se transforma en un punto, y que además, como segundo parámetro, podemos pasar datos a la vista. A continuación se incluyen algunos ejemplos:
```
return view('home');
return view('proyectos.index');
return view('proyectos.show', array('id'=>$id));
```

Una vez hechos estos cambios ya podemos probarlo en el navegador, el cual debería mostrar en todos los casos la plantilla base con la barra de navegación principal y los estilos de Bootstrap aplicados. En la sección principal de contenido de momento solo podremos ver los textos que hemos puesto de ejemplo.
