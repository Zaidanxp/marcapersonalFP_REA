# Ejercicios API

## Modificación del Proyecto para Trabajar con un Repositorio Común

Partiendo del desarrollo explicado en el fichero [Consumiendo API externas](./12_apiExterna.md), vamos a modificarlo de tal manera que, en el caso de que el proyecto no tenga asignado un valor en el atributo `url_github`, el proyecto no genere un nuevo repositorio. En lugar de eso, los ficheros se deben enviar, a través del método `pushZipFiles` de `GitHubServiceProvider`, al repositorio común definido en la variable de entorno `GITHUB_PROYECTOS_REPO`. En nuestro caso, este repositorio será `https://github.com/2DAW-CarlosIII/proyectosRepo`.

Los archivos se deben enviar tantas veces como ciclos formativos asociados al proyecto haya, creando o utilizando una estructura de archivos `/ciclo/año/ficherosProyecto`.

### Detalles del Ejercicio

1. Verificar si `url_github` está presente en el proyecto.
2. Si `url_github` no está presente:
   1. Obtener la variable de entorno `GITHUB_PROYECTOS_REPO`.
   2. Para cada ciclo formativo asociado al proyecto:
      1. Crear o utilizar la estructura de archivos `/{ciclo}/{año}/ficherosProyecto`.
      2. Utilizar el método `pushZipFiles` de `GitHubServiceProvider` para enviar los archivos al repositorio común.

### Plazos

- Fecha límite **entrega**: 14 de febrero a las 23:59.
- Fecha límite **revisión**: 15 de febrero a las 23:59.

## Copiar a repositorio común

Habrá casos en los que el proyecto lo hayan alojado los estudiantes en un repositorio propio, pero, por motivos de visibilidad, quieran alojar una copia en el repositorio común. Para ello, vamos a añadir un nuevo método en el controlador de proyectos que obtenga el fichero comprimido asociado al atributo `url_github` y lo envíe al repositorio común.

Para obtener el archivo comprimido del repositorio de GitHub, utilizaremos el método `getZipFileFromRepo` de `GitHubServiceProvider`. Este método recibe como parámetro la URL del repositorio, le concatena la cadena `/archive/refs/heads/master.zip` y lo almacena temporalmente en la carpeta `storage` como si hubiera sido subido desde la página de edición del proyecto en Reac-admin, asociándolo también al atributo `fichero` del proyecto.

Una vez obtenido el fichero, el proceso de envío al repositorio común es el mismo que el explicado en el ejercicio anterior.

### Plazos

- Fecha límite **entrega**: 14 de febrero a las 23:59.
- Fecha límite **revisión**: 15 de febrero a las 23:59.
