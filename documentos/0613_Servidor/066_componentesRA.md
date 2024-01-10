# React-admin `<Resource />`

Los componentes `<Resource>` definen las rutas _CRUD_ de una aplicación _react-admin_.

En términos de _react-admin_, un recurso es una cadena que se refiere a un tipo de entidad (como `proyectos`, `actividades` o `users`). Los registros son objetos con un campo `id`, y dos registros del mismo recurso tienen la misma estructura de campo (por ejemplo, todos los registros de `proyectos` tienen un `nombre`, un `dominio`, etc.).

Un componente `<Resource>` tiene 3 responsabilidades:

- Define las rutas _CRUD_ de un recurso determinado (para mostrar una lista de registros, los detalles de un registro o para crear uno nuevo).
- Crea un contexto que permite a todos los componentes descendientes conocer el nombre del recurso actual (este contexto se llama ResourceContext).
- Almacena la definición del recurso (su nombre, icono y etiqueta) dentro de un contexto compartido (este contexto se llama ResourceDefinitionContext).

Los componentes `<Resource>` solo se pueden utilizar como hijos del componente `<Admin>`.

## Personalizando un recurso

Para personalizar un recursos, puede utilizar las siguientes propiedades:

- `name`: el nombre del recurso. Se utiliza para construir las rutas _CRUD_ del recurso. También se utiliza para construir el nombre de los componentes relacionados con el recurso (por ejemplo, `ListGuesser` para la lista de registros, `EditGuesser` para el formulario de edición de un registro, etc.). Si no se proporciona, se utilizará el nombre del recurso en minúsculas.
- `list`: el componente que se utilizará para mostrar la lista de registros. Por defecto, se utiliza el componente `ListGuesser`.
- `create`: el componente que se utilizará para mostrar el formulario de creación de un registro. Por defecto, se utiliza el componente `CreateGuesser`.
- `edit`: el componente que se utilizará para mostrar el formulario de edición de un registro. Por defecto, se utiliza el componente `EditGuesser`.
- `show`: el componente que se utilizará para mostrar los detalles de un registro. Por defecto, se utiliza el componente `ShowGuesser`.
- `icon`: el icono que se utilizará para mostrar el recurso en el menú. Por defecto, se utiliza el componente `ListGuesser`.
- `options`: opciones adicionales para personalizar el recurso. Por defecto, se utiliza el componente `ListGuesser`.

Por ejemplo, para personalizar el recurso `proyectos`, vamo a incorporar los siguientes modificar el fichero `resources/js/react-admin/App.js`:

1. Se importan los componentes personalizados `ProyectoList`, `ProyectoEdit`, `ProyectoShow` y `ProyectoCreate` desde el archivo `proyectos.js`. Estos componentes se utilizarán para mostrar la lista, el formulario de edición, los detalles y el formulario de creación de registros respectivamente.

2. Se importa el icono personalizado `ProyectoIcon` desde `@mui/icons-material/AccountTree`. Este icono se utilizará para representar el recurso "proyectos" en el menú de la aplicación.

3. Se sustituye el `<Resource>` correspondiente al recurso `proyectos` especificando los componentes personalizados que se van a utilizar: `ProyectoList`, `ProyectoEdit` y `ProyectoShow`. También se agrega el componente `ProyectoCreate` para el formulario de creación de registros.

4. Se asigna el icono personalizado `ProyectoIcon` al recurso "proyectos" utilizando la propiedad `icon`.

Ahora, en lugar de utilizar los componentes predeterminados, se utilizan los componentes personalizados definidos en el archivo `resources/js/react-admin/pages/proyectos.jsx`. Además, se utiliza un icono personalizado para representar el recurso en el menú de la aplicación.

Para conseguirlo, debemos modificar el fichero `resources/js/react-admin/App.jsx` con los siguientes cambios:

```diff
diff --git a/resources/js/react-admin/App.tsx b/resources/js/react-admin/App.tsx
index d59478c..dbd21c4 100644
--- a/resources/js/react-admin/App.tsx
+++ b/resources/js/react-admin/App.tsx
@@ -2,16 +2,23 @@
 import { Admin, Resource, ListGuesser, EditGuesser, ShowGuesser } from 'react-admin';
 import { dataProvider } from './dataProvider';
 import { authProvider } from './authProvider';
+import { ProyectoList, ProyectoEdit, ProyectoShow, ProyectoCreate } from './pages/proyectos';
+import ProyectoIcon from '@mui/icons-material/AccountTree';
 
 export const App = () => (
     <Admin
         dataProvider={dataProvider}
                authProvider={authProvider}
        >
-        <Resource name="proyectos" list={ListGuesser} edit={EditGuesser} show={ShowGuesser} />
+        <Resource name="proyectos"
+            icon={ProyectoIcon}
+            list={ProyectoList}
+            edit={ProyectoEdit}
+            show={ProyectoShow}
+            create={ProyectoCreate}
+        />
         <Resource name="users" list={ListGuesser} edit={EditGuesser} show={ShowGuesser} />
         <Resource name="actividades" list={ListGuesser} edit={EditGuesser} show={ShowGuesser} />
     </Admin>
 );
``` 

Como hemos comentado anteriormente, los componentes personalizados se importan desde el archivo `resources/js/react-admin/pages/proyectos.jsx`, cuyo contenido se facilita a continuación:

```javascript
// in resources/js/react-admin/pages/proyectos.jsx
import {
    List,
    SimpleList,
    Datagrid,
    TextField,
    ReferenceField,
    EditButton,
    Edit,
    Create,
    SimpleForm,
    ReferenceInput,
    TextInput,
    NumberField,
    NumberInput,
    FunctionField,
    SelectInput,
    ShowButton,
    Show,
    SimpleShowLayout
  } from 'react-admin';

import { useRecordContext} from 'react-admin';
import { useMediaQuery } from '@mui/material';

const TutorInput = () => (
    <ReferenceInput label="Tutor" source="docente_id" reference="users">
        <SelectInput
        label="Tutor"
        source="docente_id"
        optionText={record => record && `${record.nombre} ${record.apellidos}`} />
    </ReferenceInput>
)
const proyectosFilters = [
    <TextInput source="q" label="Search" alwaysOn />,
    TutorInput(),
];

export const ProyectoList = () => {
  const isSmall = useMediaQuery((theme) => theme.breakpoints.down('sm'));
  return (
    <List filters={proyectosFilters} >
      {isSmall ? (
        <SimpleList
          primaryText="%{nombre}"
          secondaryText="%{dominio}"
          tertiaryText="%{calificacion}"
          linkType={(record) => (record.canEdit ? 'edit' : 'show')}
        >
          <EditButton />
        </SimpleList>
      ) : (
        <Datagrid bulkActionButtons={false} >
          <TextField source="id" />
          <TextField source="nombre" />
          <TextField source="dominio" />
          <ReferenceField label="Tutor" source="docente_id" reference="users">
            <FunctionField render={record => record && `${record.nombre} ${record.apellidos}`} />
          </ReferenceField>
          <NumberField source="calificacion" />
          <ShowButton />
          <EditButton />
        </Datagrid>
      )}
    </List>
  );
}

export const ProyectoTitle = () => {
  const record = useRecordContext();
  return <span>Proyecto {record ? `"${record.nombre}"` : ''}</span>;
};

export const ProyectoEdit = () => (
    <Edit title={<ProyectoTitle />}>
    <SimpleForm>
        <TextInput source="id" disabled />
        <TextInput source="nombre" />
        <TextInput source="dominio" />
        <TutorInput />
        <NumberInput source="calificacion" />
    </SimpleForm>
    </Edit>
);

export const ProyectoShow = () => (
    <Show>
        <SimpleShowLayout>
            <TextField source="id" />
            <TextField source="nombre" />
            <TextField source="dominio" />
            <ReferenceField label="Tutor" source="docente_id" reference="users">
                <FunctionField render={record => record && `${record.nombre} ${record.apellidos}`} />
            </ReferenceField>
            <NumberField source="calificacion" />
        </SimpleShowLayout>
    </Show>
);

export const ProyectoCreate = () => (
    <Create>
        <SimpleForm>
            <TextInput source="nombre" />
            <TextInput source="dominio" />
            <TutorInput />
            <NumberInput source="calificacion" />
        </SimpleForm>
    </Create>
);
```

El código anterior define cuatro componentes de _React_ que se utilizan para personalizar la interfaz de usuario de la administración de `proyectos` en nuestra aplicación. Estos componentes son `ProyectoList`, `ProyectoEdit`, `ProyectoShow` y `ProyectoCreate`.

1. `ProyectoList`: Este componente se utiliza para **mostrar la lista** de proyectos. Utiliza el componente [`<List>`](https://marmelab.com/react-admin/List.html) de _React Admin_ y dentro de él, el componente [`<Datagrid>`](https://marmelab.com/react-admin/Datagrid.html). Dentro del `<Datagrid>`, se definen varios componentes [`<TextField>`](https://marmelab.com/react-admin/TextField.html) y [`<ReferenceField>`](https://marmelab.com/react-admin/ReferenceField.html) para mostrar los campos de cada proyecto en la lista.

2. `ProyectoEdit`: Este componente se utiliza para **editar** un proyecto existente. Utiliza el componente [`<Edit>`]() de _React Admin_ y dentro de él, el componente [`<SimpleForm>`](https://marmelab.com/react-admin/SimpleForm.html). Dentos del `<SimpleForm>`, se definen varios componentes [`<TextInput>`](https://marmelab.com/react-admin/TextInput.html), `<TutorInput>` y [`<NumberInput>`](https://marmelab.com/react-admin/NumberInput.html) para editar los campos de un proyecto.
    2.1 El componente [TutorInput] se genera anteriormente con el siguiente código, que utiliza un [`ReferenceInput`](https://marmelab.com/react-admin/ReferenceInput.html):
    ```javascript
    const TutorInput = () => (
        <ReferenceInput label="Tutor" source="docente_id" reference="users">
            <SelectInput
            label="Tutor"
            source="docente_id"
            optionText={record => record && `${record.nombre} ${record.apellidos}`} />
        </ReferenceInput>
    )
    ```

3. `ProyectoShow`: Este componente se utiliza para mostrar los detalles de un proyecto. Utiliza el componente [`<Show>`]() de React Admin y dentro de él, el componente [`<SimpleShowLayout>`](). Dentro del `<SimpleShowLayout>`, también utiliza los componentes `<TextField>`, `<ReferenceField>` y `<NumberField>` usados en `ProyectoList`.

4. `ProyectoCreate`: Este componente se utiliza para crear un nuevo proyecto. Utiliza el componente `<Create>` de React Admin y dentro de él, el componente `<SimpleForm>`. Dentro del `<SimpleForm>`, también utiliza los componentes `<TextInput>`, `<TutorInput>` y `<NumberInput>` usados en `ProyectProyectoEditoList`.

Cada uno de estos componentes se exporta para que puedan ser importados y utilizados por el fichero `resources/js/react-admin/App.jsx`.
