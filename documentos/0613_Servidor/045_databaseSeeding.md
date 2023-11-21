4.5. Inicialización de la base de datos (Database Seeding)

Laravel también facilita la inserción de datos iniciales o datos semilla en la base de datos. Esta opción es muy útil para tener datos de prueba cuando estamos desarrollando una web o para crear tablas que ya tienen que contener una serie de datos en producción.

Los ficheros de "semillas" se encuentran en la carpeta database/seeders. Por defecto Laravel incluye el fichero DatabaseSeeder con el siguiente contenido:

<?php

use Illuminate\Database\Seeder;
use Illuminate\Database\Eloquent\Model;

class DatabaseSeeder extends Seeder
{
    /**
     * Run the database seeds.
     * @return void
     */
    public function run()
    {
        //...
    }
}

Al lanzar la incialización se llamará por defecto al método run de la clase DatabaseSeeder. Desde aquí podemos crear las semillas de varias formas:

    Escribir el código para insertar los datos dentro del propio método run.
    Crear otros métodos dentro de la clase DatabaseSeeder y llamarlos desde el método run. De esta forma podemos separar mejor las inicializaciones.
    Crear otros ficheros Seeder y llamarlos desde el método run es la clase principal.

Según lo que vayamos a hacer nos puede interesar una opción u otra. Por ejemplo, si el código que vamos a escribir es poco nos puede sobrar con las opciones 1 o 2, sin embargo si vamos a trabajar bastante con las inicializaciones quizás lo mejor es la opción 3.

A continuación se incluye un ejemplo de la opción 1:

<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

class DatabaseSeeder extends Seeder
{
    /**
     * Run the database seeders.
     *
     * @return void
     */
    public function run()
    {
        DB::table('users')->insert([
            'name' => Str::random(10),
            'email' => Str::random(10).'@gmail.com',
            'password' => Hash::make('password'),
        ]);
    }
}

Como se puede ver en el ejemplo en general tendremos que eliminar primero los datos de la tabla en cuestión y posteriormente añadir los datos. Para insertar datos en una tabla podemos utilizar el "Constructor de consultas" y "Eloquent ORM".
Crear ficheros semilla

Como hemos visto en el apartado anterior, podemos crear más ficheros o clases semilla para modularizar mejor el código de las inicializaciones. De esta forma podemos crear un fichero de semillas para cada una de las tablas o modelos de datos que tengamos.

En la carpeta database/seeders podemos añadir más ficheros PHP con clases que extiendan de Seeder para definir nuestros propios ficheros de "semillas". El nombre de los ficheros suele seguir el mismo patrón <nombre-tabla>TableSeeder, por ejemplo "UsersTableSeeder". Artisan incluye un comando que nos facilitará crear los ficheros de semillas y que además incluirán las estructura base de la clase. Por ejemplo, para crear el fichero de inicialización de la tabla de usuarios haríamos:

php artisan make:seeder UsersTableSeeder

Para que esta nueva clase se ejecute tenemos que llamarla desde el método run de la clase principal DatabaseSeeder de la forma:

class DatabaseSeeder extends Seeder 
{
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Support\Facades\Schema;

    public function run()
    {
        Model::unguard();
        Schema::disableForeignKeyConstraints();

        $this->call(UsersTableSeeder::class);

        Model::reguard();

        Schema::enableForeignKeyConstraints();
 }
}

El método call lo que hace es llamar al método run de la clase indicada. Además en el ejemplo hemos añadido las llamadas a unguard y a reguard, que lo que hacen es desactivar y volver a activar (respectivamente) la inserción de datos masiva o por lotes.
Ejecutar la inicialización de datos

Una vez definidos los ficheros de semillas, cuando queramos ejecutarlos para rellenar de datos la base de datos tendremos que usar el siguiente comando de Artisan:

php artisan db:seed
