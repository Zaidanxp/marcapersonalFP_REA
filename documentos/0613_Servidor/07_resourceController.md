# Controladores de Recursos

## 1. Introducción

Un controlador de recursos **API** en _Laravel_ es similar a un controlador de recursos regular, pero está diseñado específicamente para manejar las solicitudes _API_. No incluye los métodos `create` y `edit` porque en una _API_ típicamente no se manejan formularios _HTML_. Aquí está la correspondencia de los métodos para un controlador de recursos _API_:

- `index`: Maneja una solicitud GET para recuperar todos los recursos.
- `store`: Maneja una solicitud POST para crear un nuevo recurso.
- `show`: Maneja una solicitud GET para mostrar un recurso específico.
- `update`: Maneja una solicitud PUT o PATCH para actualizar un recurso existente.
- `destroy`: Maneja una solicitud DELETE para eliminar un recurso existente.

Estos métodos corresponden a las operaciones **CRUD** (Crear, Leer, Actualizar, Borrar) en una **API RESTful**.

### Migración y Seeder

Los capítulos posteriores requieren la utilización de los ficheros de **migraciones** y de **semillas** correspondientes a la tablas `ciclos` y `familias profesionales`. Estos ficheros se facilitan en los materiales:

- migraciones
    - [familias profesionales](./materiales/ejercicios-laravel/2024_01_11_165855_familias_profesionales_create_table.php)
    - [ciclos formativos](./materiales/ejercicios-laravel/2024_01_11_170253_ciclos_create_table.php)
- _seeders_
    - [familias profesionales](./materiales/ejercicios-laravel/FamiliasProfesionalesSeeder.php)
    - [ciclos formativos](./materiales/ejercicios-laravel/CiclosSeeder.php)

> La fecha de los fichero de migraciones habrá que actualizarla, teniendo en cuenta que debe tener una fecha anterior el fichero de familias profesionales que el de ciclos formativos.

> La ejecución de los ficheros de semillas deberá ser invocada desde el fichero `database/seeders/DatabaseSeeder.php`.
