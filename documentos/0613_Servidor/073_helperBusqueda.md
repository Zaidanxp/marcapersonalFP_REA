# Utilizar el cuadro de búsqueda

## Filtrando a través de las búsquedas

El _frontend_ dispone de un cuadro de búsqueda con el que poder filtrar los registros que deben mostrarse. Hasta ahora, no hemos hecho uso de ese parámetro de la petición en el _backend_.

Para poder utilizarlo, debemos inyectar el objeto `Request` en el método `index()` del controlador. Esto nos permitirá acceder a todos los parámetros de la petición.

```php
    public function index(Request $request)
```

El _frontend_ envía el contenido de los cuadros de búsqueda en el parámetro `filter`. Este parámetro es un **array asociativo**, en el _backend_ sustituiremos la búsqueda inicial por el valor de ese parámetro:

```php
        $busqueda = $request->input('filter');
```

Por defecto, el _frontend_ nos envía el primer cuadro de búsqueda en el elemento `q` del _array_ anterior, por lo que si queremos hacer una búsqueda de los `ciclos` cuyo `nombre` contiene la clave de búsqueda, deberemos codificar el método `index()` como se muestra a continuación:

```php
    public function index(Request $request)
     {
        $busqueda = $request->input('filter');
        $registrosCiclos =
        ($busqueda && array_key_exists('q', $busqueda))
            ? Ciclo::where('nombre', 'like', '%' .$busqueda['q'] . '%')->get()
            : Ciclo::all();

            return CicloResource::collection($registrosCiclos);
        }
```

La búsqueda que hemos implementado en `CicloController` también la podemos implementar en `UserController`. Vamos, además, a permitir que la búsqueda se realice en una serie de atributos seleccionados y almacenados en un _array_. El código del método `index()` de `UserController` quedaría como el siguiente:

```php
    public function index(Request $request)
    {
        $busquedaArray = [
            'nombre',
            'apellidos',
            'email',
        ];
        $busquedaFiltroQ = $request->input('q');
        $registrosUsuario = User::query();
        if($busquedaFiltroQ) {
            foreach ($busquedaArray as $fieldName) {
                $registrosUsuario = $registrosUsuario
                    ->orWhere($fieldName, 'like', '%' .$busquedaFiltroQ . '%');
            }
        }

        return UserResource::collection(
            $registrosUsuario->orderBy($request->_sort ?? 'id', $request->_order ?? 'asc')
            ->paginate($request->perPage)
        );
    }
```

## Refactorizando en un helper

Tras haber utilizado Query Builder para generar consultas con el parámetro filter enviado desde el frontend, nos damos cuenta de que podemos extender esta funcionalidad a todos los controladores. Para ello, sería conveniente crear un helper de Laravel para generar esas consultas, al que le deberíamos enviar, como parámetros, el array de atributos sobre los que buscar y la clase del modelo que debemos utilizar para generar ese Query Builder.

Para crear un helper en Laravel, realizaremos los siguientes pasos

    Crear un directorio Helpers en la carpeta app.

    mkdir app/Helpers

    En el directorio Helpers vamos a crear un archivo searchByFields.php con el siguiente contenido:

    <?php

    function searchByField($fieldsArray, $modelClass){
        $busquedaFiltroQ = request()->input('filter');
        $query = $modelClass::query();
        if($busquedaFiltroQ && array_key_exists('q', $busquedaFiltroQ)) {
            foreach ($fieldsArray as $fieldName) {
                $query = $query
                    ->orWhere($fieldName, 'like', '%' .$busquedaFiltroQ['q'] . '%');
            }
        }
        return $query;
    }

    En el fichero composer.json añadiremos el fichero searchByFields.php al atributo files de la propiedad autoload para que Composer sea automáticamente cargada cuando arranque la aplicación. Esto es realizado por Composer usando el estándar PSR-4 auto-loading.

        "autoload": {
            "psr-4": {
                "App\\": "app/",
                "Database\\Factories\\": "database/factories/",
                "Database\\Seeders\\": "database/seeders/"
            },
            "files": [
               "app/Helpers/searchByFields.php"
            ]
        },

    Por último, ejecutaremos composer dump-autoload para refrescar la cache de autoload.

Tras crear el helper searchByField() el código del método index de UserController quedaría de la siguiente forma:
    public function index(Request $request)
    {
        $numElementos = $request->input('numElements');

        $registros = searchByField(array('name', 'email'), User::class);

        return UserResource::collection($registros->paginate($numElementos));
    }

Mientras que el de CustomerController quedaría:
    public function index(Request $request)
    {
        $numElementos = $request->input('numElements');

        $registros = searchByField(array('first_name', 'last_name', 'job_title', 'city', 'country'), Customer::class);

        return CustomerResource::collection($registros->paginate($numElementos));
    }
