# Manejar Ficheros en la API

Como comentamos en el capítulo dedicado a los [datos de entrada](./051_datosEntrada.md#ficheros-de-entrada) y en el capítulo [utilización de ficheros](./0512_utilizarFicheros.md), _Laravel_ facilita el manejo de ficheros, tanto si se van a almacenar en el sistema de ficheros _local_, como si se van a almacenar en un servicio como _Amazon S3_.

En esta práctica guiada, vamos a utilizar el driver que incluye para el almacenamiento en el sistema de archivos `public` para conseguir subir al servidor un archivo comprimido asociado al proyecto, que posteriormente enviaremos a GitHub para su despliegue.

## Backend

### Modelo `Proyecto`

Para poder manejar el atributo `fichero` en operaciones masivas, debemos modificar la propiedad `fillable`del modelo `app/Models/Proyecto.php`, añadiendo el atributo `fichero` al _array_:

```php
    protected $fillable = [
        'nombre',
        'docente_id',
        'dominio',
        'metadatos',
        'calificacion',
        'fichero',
    ];
```

### Controlador de proyectos

Ya disponemos de un controlador de proyectos para gestionar los _endpoints_ de ese tipo de recurso. No obstante, vamos a ampliar su funcionalidad para que acepte un fichero asociado a la ruta `PUT /proyectos/{proyecto}`, por lo que tendremos que modificar el `app/Http/Controllers/API/ProyectoController.php` con el siguiente contenido:

```php
    public function update(Request $request, Proyecto $proyecto)
    {
        $proyectoData = $request->all();
        if($proyectoRepoZip = $request->file('fichero')) {
            $path = $proyectoRepoZip->store('public/repoZips');
            $proyectoData['fichero'] = $path;
        }
        $proyecto->update($proyectoData);

        return new ProyectoResource($proyecto);
    }
```

Con los cambios anteriores, ya tendríamos preparado el Backend.

## Frontend

### Página Proyectos

En la página de React-admin correspondiente a los proyectos, añadiremos un componente `<FileInput />` que permitirá enviar el fichero del proyecto.

Para ello, haremos las siguientes modificaciones en el archivo:

```jsx
    SelectInput,
+    FileInput, FileField,
    ShowButton,
...
} from 'react-admin';
            
            <NumberInput source="calificacion" />
+            <FileInput source="fichero" label="Archivo comprimido con el proyecto">
+                <FileField source="src" title="title" />
+            </FileInput>
        </SimpleForm>
    </Edit>
```

## DataProvider

Hasta el momento, las peticiones `PUT` realizadas por el `DataProvider` de React-admin han funcionado correctamente. Pero, la función `update`del `DataProvider` no funciona si hay que enviar ficheros, siendo necesario ampliar las funcionalidades del `DataProvider` utilizado por React-admin.

Para ello, vamos a modficar el fichero `resources/js/react-admin/dataProvider.ts` para añadir esas funcionalidades extendidas que necesitamos. El código resultante será el que se muestra a continuación:

```ts
// en resources/js/react-admin/dataProvider.ts
import jsonServerProvider from 'ra-data-json-server';
import { stringify } from 'query-string';
import { fetchUtils } from 'ra-core';

const apiUrl = import.meta.env.VITE_JSON_SERVER_URL;

const dataProvider = jsonServerProvider(
    apiUrl
);

const httpClient = (url, options = {}) => {
    return fetchUtils.fetchJson(url, options);
};

dataProvider.getMany = (resource, params) => {
    const query = {
        id: params.ids,
    };
    const url = `${apiUrl}/${resource}?${stringify(query, {arrayFormat: 'bracket'})}`;
    return httpClient(url).then(({ json }) => ({ data: json }));
}

dataProvider.update = (resource, params) => {
    if (resource !== 'proyectos' || !params.data.fichero) {
        return dataProvider.update(resource, params);
    }

    let formData = new FormData();
    for (const property in params.data) {
        formData.append(`${property}`, `${params.data[property]}`);
    }

    formData.append('fichero', params.data.fichero.rawFile)
    formData.append('_method', 'PUT')

    const url = `${apiUrl}/${resource}/${params.id}`
    return httpClient(url, {
        method: 'POST',
        body: formData,
    })
    .then(json => {
        return {
            ...json,
            data: json.json
        }
    })
}

export { dataProvider };

```

Como podemos observar, si la petición no se refiere a la actualización de los proyectos o no incluye un fichero, se envía al `dataProvider` básico. En caso contrario, se genera una petición `PUT` haciendo que los datos enviados viajen en el `body` de la petición como `formData`.
